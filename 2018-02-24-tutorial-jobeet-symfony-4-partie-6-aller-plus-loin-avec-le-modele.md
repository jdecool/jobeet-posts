---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 6: Aller plus loin avec le modèle"
menu: blog
priority: 0.80
type: jobeet
theme: php
github: "https://github.com/jdecool/jobeet-posts"
---

Notre application Jobeet commence à devenir utilisable. Nous savons maintenant créer des pages, les afficher et naviguer entre elles en utilisant le framework Symfony via les différents composants qui sont à notre disposition. Attardons-nous un peu sur la couche [modèle]({% post_url 2017-09-20-tutorial-jobeet-symfony-4-partie-3a-le-modele-de-donnees %}) de notre projet.

Cette dernière est actuellement composée de nos entités (les classes qui représentent les données stockées en base). Nous allons dans ce chapitre, travailler sur l'optimisation de notre code, ce qui vous permettra d'en apprendre un peu plus sur le sujet.

Revenons sur nos différents [scénarios]({% post_url 2017-09-19-tutorial-jobeet-symfony-4-partie-2-le-projet %}) et plus précisément sur le scénario *F1*: `En tant qu'utilisateur, je vois les dernières offres actives sur la page d'accueil`. Car si vous avez bien suivi ce que nous avons réalisé, la page d'accueil liste actuellement toutes les offres d'emploi aussi bien celles qui sont actives que celles qui ne le sont pas.

Un emploi est considéré actif s'il a été posté il y a moins de 30 jours. Commençons par modifier la requête effectuée dans la méthode `JobController::index` :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

// ...
use DateTime;

class JobController extends AbstractController
{
    public function index(EntityManagerInterface $em): Response
    {
        $queryBuilder = $em->getRepository(Job::class)->createQueryBuilder('j');
        $queryBuilder->andWhere('j.createdAt > :date');
        $queryBuilder->setParameter('date', new DateTime('-30 day'));
        $jobs = $queryBuilder->getQuery()->getResult();

        return $this->render('job/index.html.twig', [
            'jobs' => $jobs,
        ]);
    }
}
{% endhighlight %}

> Pour écrire une requête "complexe", nous utilisons l'objet {% ext `QueryBuilder`|http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html%} fourni par Doctrine et qui permet comme son nom l'indique de créer une requête compréhensible par l'ORM sans devoir écrire de code SQL. L'avantage est que Doctrine adaptera la requête au type de base de données avec lequel il communique (SQLite, MySQL, PostgresSQL, ...). L'inconvénient est que cela masque complètement la requête qui est générée.

Notre requête commence à devenir plus complexe et il devient intéressant de l'extraire de notre contrôleur pour que cette dernière puisse être réutilisée sans devoir dupliquer le code. Jusqu'à maintenant, la méthode `EntityManager::getRepository` nous permettait d'obtenir un objet générique que Doctrine utilise pour fournir des méthodes de base permettant de faire des requêtes en base de données.

Nous allons maintenant définir une classe de type `Repository`. Les objets de types `Repository` contiennent des méthodes permettant de récupérer des données en base. Définissons donc une classe `JobRepository`, qui comme son nom l'indique, nous permettra de récupérer des données liées aux offres d'emploi.

{% highlight php %}
<?php // src/Repository/JobRepository.php

namespace App\Repository;

use DateTime;
use Doctrine\ORM\EntityRepository;

class JobRepository extends EntityRepository
{
    public function findActive(DateTime $date)
    {
        return $this->createQueryBuilder('j')
            ->andWhere('j.createdAt > :date')
            ->setParameter('date', $date)
            ->getQuery()
            ->getResult();
    }
}
{% endhighlight %}

Une fois notre classe créée, nous allons devoir modifier la configuration du mapping de l'entité `Job` pour indiquer à Doctrine la classe que l'ORM devra utiliser pour accéder aux données. Cela se passe dans le fichier `config/doctrine/mapping/Job.orm.yml`.

{% highlight yaml %}
# config/doctrine/mapping/Job.orm.yml
App\Entity\Job:
    type: entity
    repositoryClass: App\Repository\JobRepository

    # ...
{% endhighlight %}

Pour finir, supprimons le code du contrôleur pour utiliser notre nouvelle classe :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

// ...

class JobController extends AbstractController
{
    public function index(EntityManagerInterface $em): Response
    {
        $jobs = $em->getRepository(Job::class)->findActive(new DateTime('-30 day'));

        return $this->render('job/index.html.twig', [
            'jobs' => $jobs,
        ]);
    }
}
{% endhighlight %}

Et voilà, nous utilisons maintenant une classe pour la récupération des données de notre offre d'emploi, ce qui permet d'isoler le code dédié à la récupération des données et permet ainsi d'améliorer la maintenabilité de notre projet.

> Il est possible de consulter la requête générée par Doctrine en consultant les logs générés par l'application. Par défaut, Symfony crée les logs sur la sortie standard et son donc consultable directement sur le terminal.

{% highlight plaintext %}
2018-02-10T19:32:20+01:00 [debug] SELECT j0_.id AS id_0, j0_.type AS type_1, j0_.company AS company_2, j0_.logo AS logo_3, j0_.url AS url_4, j0_.position AS position_5, j0_.location AS location_6, j0_.description AS description_7, j0_.how_to_apply AS how_to_apply_8, j0_.token AS token_9, j0_.is_public AS is_public_10, j0_.is_activated AS is_activated_11, j0_.email AS email_12, j0_.expires_at AS expires_at_13, j0_.created_at AS created_at_14, j0_.updated_at AS updated_at_15, j0_.category_id AS category_id_16 FROM job j0_ WHERE j0_.created_at > ?
{% endhighlight %}

Un moyen plus simple d'accéder à ces informations est d'installer le composant `symfony/profiler-pack`.

{% highlight bash %}
$ composer require symfony/profiler-pack --dev
{% endhighlight %}

Ce dernier permet la mise en place d'une interface graphique qui affiche un certain nombre d'informations sur votre application. Chaque bundle peut ainsi y afficher des données. Cette interface ajoute une barre permettant d'avoir un résumé des informations disponibles :

{% imgFull profiler-bar.png %}

Il est également possible d'accéder à un détail des informations récupérées en cliquant sur l'icône du composant concerné :

{% imgFull profiler-detail.png %}

Pour l'heure, notre code est encore loin d'être parfait. Pour récupérer toutes les offres actives, nous sommes systématiquement obligés de passer en paramètre la date à partir de laquelle les offres sont visibles. Cela revient à dupliquer le calcul de la date et serait source d'erreurs. Pour corriger ce problème, nous allons créer une constante qui nous permettra de masquer ce calcul. Et pour optimiser notre requête, nous allons utiliser le champ `expiresAt` afin de stocker la date d'expiration d'une offre plutôt que de devoir la calculer.

Tout comme pour les dates de création et de modification de nos offres d'emploi, nous allons utilisons le gestionnaire d'événement de Doctrine pour mettre à jour la valeur du champ automatiquement. Commençons par ajouter le code nécessaire à notre entité :

{% highlight php %}
<?php // src/Entity/Job.php

namespace App\Entity;

// use ...

class Job
{
    public const OFFER_LIFETIME = 30; // durée de vie d'une offre en jours

    // ...

    public function setExpiresAtValue(LifecycleEventArgs $event): self
    {
        // nous remplissons automatiquement la date d'expiration si cette dernière n'a pas été saisie
        // manuellement
        if (!$this->expiresAt) {
            $this->expiresAt = new DateTime('+'.self::OFFER_LIFETIME.' day');
        }

        return $this;
    }
}
{% endhighlight %}

N'oublions pas d'ajouter la configuration liée à cette gestion d'événement.

{% highlight yaml %}
# config/doctrine/mapping/Job.orm.yml
App\Entity\Job:
    # ...
    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue, setExpiresAtValue ]
{% endhighlight %}

Nous pouvons maintenant mettre à jour notre requête :

{% highlight php %}
<?php // src/Repository/JobRepository.php

namespace App\Repository;

use DateTime;
use Doctrine\ORM\EntityRepository;

class JobRepository extends EntityRepository
{
    public function findActive()
    {
        return $this->createQueryBuilder('j')
            ->andWhere('j.expiresAt >= :date')
            ->setParameter('date', new DateTime())
            ->getQuery()
            ->getResult();
    }
}
{% endhighlight %}

> N'oubliez pas d'enlever le paramètre dans l'appel de la méthode dans la classe `JobController::index`

Le code de notre application est maintenant plus simple et plus maintenable, mais si nous voulons pouvoir tester que tout fonctionne correctement, encore faut-il mettre à jour nos données de test. Car dans les données actuelles, nous avons défini une date d'expiration des offres au 10/10/2012 et nous ne voyons donc maintenant plus aucune offre. Supprimons les dates d'expiration de notre jeu actuel et ajoutons une offre expirée ({% ext consultez directement ce fichier pour avoir le code correspondant|https://github.com/jdecool/jobeet/blob/06-modele/src/DataFixtures/JobFixtures.php %}).

En rechargeant les fixtures au travers de la commande `bin/console doctrine:fixtures:load`, l'affichage ne devrait pas avoir changé, mais si vous regardez les données en base, vous constaterez qu'il y a pourtant bien 3 offres d'emploi enregistrées.

Si nous revenons à nos [scénarios utilisateurs]({% post_url 2017-09-19-tutorial-jobeet-symfony-4-partie-2-le-projet %}), nous avons spécifié que les offres devaient être classées par catégories, ce qui n'est actuellement pas le cas. Pour répondre à ce besoin, nous allons créer une classe de type `Repository` pour notre entité `Category`. Cette dernière nous permettra de lister les catégories existantes avec les offres d'emploi correspondantes.

Commençons par modifier le mapping Doctrine :

{% highlight yaml %}
# config/doctrine/mapping/Job.orm.yml
App\Entity\Category:
    type: entity
    repositoryClass: App\Repository\CategoryRepository

    # ...
{% endhighlight %}

Créons maintenant la classe correspondante avec la nouvelle méthode de récupération des offres par catégorie :

{% highlight php %}
<?php // src/Entity/CategoryRepository.php

declare(strict_types=1);

namespace App\Repository;

use DateTime;
use Doctrine\ORM\EntityRepository;

class CategoryRepository extends EntityRepository
{
    public function findCategoriesWithJobs()
    {
        return $this->createQueryBuilder('c')
            ->join('c.jobs', 'j')
            ->where('j.expiresAt >= :date')
            ->setParameter('date', new DateTime())
            ->getQuery()
            ->getResult();
    }
}
{% endhighlight %}

> La requête Doctrine a été construite via le `QueryBuilder`. Doctrine implémente également son propre langage de requête appelé DQL (dérivé du SQL). Le `QueryBuilder` tout comme le `DQL` se base sur nos entités pour construire les requêtes effectuées en base de données, cela permet ensuite à Doctrine de créer les objets correspondants.

Utilisons maintenant cette dernière dans l'affichage de notre homepage :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

// ...

class JobController extends AbstractController
{
    public function index(EntityManagerInterface $em): Response
    {
        $categories = $em->getRepository(Category::class)->findCategoriesWithJobs();

        $jobsCategories = [];
        foreach ($categories as $category) {
            $jobsCategories[$category->getName()] = $em->getRepository(Job::class)->findActiveByCategory($category);
        }

        return $this->render('job/index.html.twig', [
            'categories' => $jobsCategories,
        ]);
    }
}
{% endhighlight %}

Ajoutons la méthode permettant de récupérer les offres actives d'une catégorie :

{% highlight php %}
<?php // src/Entity/JobRepository.php

declare(strict_types=1);

namespace App\Repository;

use DateTime;
use Doctrine\ORM\EntityRepository;

class JobRepository extends EntityRepository
{
    public function findActiveByCategory(Category $category)
    {
        return $this->createQueryBuilder('j')
            ->where('j.category = :category')
            ->andWhere('j.expiresAt >= :date')
            ->setParameter('category', $category)
            ->setParameter('date', new DateTime())
            ->getQuery()
            ->getResult();
    }
}
{% endhighlight %}

Modifions ensuite le template en conséquence :

{% highlight html %}{% raw %}
{# templates/job/index.html #}
{% extends "base.html.twig" %}

{% block body %}
    <h1 class="my-4">Liste des offres</h1>

    {% for category, jobs in categories %}
        <div class="row">
            <h2 style="font-weight: bold; margin: 2rem 0;">{{ category }}</h2>

            {% for job in jobs %}
                <div class="row">
                    <div class="col-md-7">
                        <a href="#">
                            <img class="img-fluid rounded mb-3 mb-md-0" src="{{ asset('images/' ~ job.logo) }}" alt="{{ job.company }}">
                        </a>
                    </div>
                    <div class="col-md-5">
                        <h3>{{ job.position }}</h3>
                        <p>{{ job.description }}</p>
                        <p>Posted on {{ job.createdAt|date("m/d/Y") }}</p>
                        <a class="btn btn-primary" href="{{ path('job_show', { 'id': job.id, 'company': job.companySlug, 'location': job.locationSlug, 'position': job.positionSlug }) }}">See more</a>
                    </div>
                </div>

                <hr>
            {% endfor %}
        </div>
    {% endfor %}

    <ul class="pagination justify-content-center">
        <li class="page-item disabled">
            <a class="page-link" href="#" aria-label="Previous">
                <span aria-hidden="true">&laquo;</span>
                <span class="sr-only">Previous</span>
            </a>
        </li>
        <li class="page-item">
            <a class="page-link" href="#">1</a>
        </li>
        <li class="page-item disabled">
            <a class="page-link" href="#" aria-label="Next">
                <span aria-hidden="true">&raquo;</span>
                <span class="sr-only">Next</span>
            </a>
        </li>
    </ul>
{% endblock %}
{% endraw %}{% endhighlight %}

Pour finir, nous allons sécuriser la page de consultation des offres. Effectivement, si vous connaissez l'URL d'une offre, il est possible d'accéder à cette dernière, et ce, même si la date d'expiration est dépassée. Pour cela, rajouter un contrôle dans notre action d'affichage :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

// ...
use DateTime;

class JobController extends AbstractController
{
    public function show(EntityManagerInterface $em, int $id, string $company, string $location, string $position) : Response
    {
        // dans un projet réel, il sera nécessaire de faire une requête permettant de vérifier que tous les éléments
        // correspondent à une offre d'emploi valide
        $job = $em->getRepository(Job::class)->find($id);
        if (null === $job) {
            throw new NotFoundHttpException();
        }

        $currentDate = new DateTime();
        if ($job->getExpiresAt() < $currentDate) {
            throw new NotFoundHttpException();
        }

        return $this->render('job/show.html.twig', [
            'job' => $job,
        ])
}
{% endhighlight %}

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}). Le code source de cette application est également disponible sur {% ext Github|https://github.com/jdecool/jobeet/tree/06-modele %}. Vous trouvez une branche associée à l'état du projet après chaque chapitre.
