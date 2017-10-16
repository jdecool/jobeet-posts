---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 5: Les routes"
menu: blog
priority: 0.80
type: jobeet
github: "https://github.com/jdecool/jobeet-posts"
---

Vous connaissez maintenant le principe d'une architecture MVC et comment cette dernière se met en place au sein d'un projet Symfony. Nous avons également rapidement évoqué le principe du routage (ou **routing** en anglais). Ce chapitre sera entièrement consacré à ce dernier point.

Dans un contexte web, une URL (*U*niform *R*esource *L*ocator) désigne un contenu accessible. Lorsque vous accédez à une page au travers de son URL, vous demandez au navigateur d'aller cherche un contenu identifié. C'est cette information que nous allons devoir décrire dans notre projet.

Dans la section consacrée au [contrôleur et à la vue]({% post_url 2017-09-29-tutorial-jobeet-symfony-4-partie-4a-le-controleur-et-la-vue %}), nous avons commencé à modifier le fichier `config/routes.yaml` qui permet de faire le lien entre l'URL courante du navigateur et l'action de notre contrôleur devant être exécutée. Nous ne sommes pas rentré dans les détails, mais une route est définie par un nom, un schéma d'URL ainsi que le contrôleur avec la méthode associée.

> C'est le composant {% ext `Routing`|http://symfony.com/components/Routing %} de Symfony qui s'occupe d'établir la correspondance entre notre configuration et le code de notre projet. Signalons que si deux routes possèdent le même nom, la seconde route écrase la première. De ce fait la première configuration ne sera jamais prise en compte.

Dans l'état actuel de notre projet, notre fichier `config/routes.yaml` contient la configuration suivante :

{% highlight yaml %}
# config/routes.yaml
index:
    path: /
    defaults: { _controller: 'App\Controller\JobController::index' }

# Depends on sensio/framework-extra-bundle, doctrine/annotations, and doctrine/cache
#   install with composer req sensio/framework-extra-bundle annot
#controllers:
#    resource: ../src/Controller/
#    type: annotation
{% endhighlight %}

Une seule route y est déclarée. Cette dernière correspond à la page d'accueil de notre application qui liste les offres d'emploi disponible. Ajoutons maintenant une route qui permettra d'afficher le détail d'une offre.

Les offres d'emploi sont créées dynamiquement par un utilisateur. Pour accéder au détail d'une offre, nous allons faire référence à son identifiant unique de base de données (autrement dit, la propriété `$id` de notre classe `Job`). Nous allons créer une route qui contiendra un paramètre dynamique permettant de récupérer cette information et de la transmettre à notre action.

Modifions le fichier de configuration en conséquence :

{% highlight yaml %}
# config/routes.yaml
index:
    path: /
    defaults: { _controller: 'App\Controller\JobController::index' }

job_show:
    path: /job/{id}
    defaults: { _controller: 'App\Controller\JobController::show' }

# Depends on sensio/framework-extra-bundle, doctrine/annotations, and doctrine/cache
#   install with composer req sensio/framework-extra-bundle annot
#controllers:
#    resource: ../src/Controller/
#    type: annotation
{% endhighlight %}

Nous venons donc de rajouter une nouvelle route nommée `job_show` qui correspond à l'appel de la méthode `show` de notre classe `JobController`. Le paramètre de la route est indiqué entre crochet (`{id}`), ce dernier correspond au nom de la variable qui sera automatiquement passé à la méthode du contrôleur.

> Sur notre environnement de développement local, il sera possible d'accéder à une offre d'emploi au travers de l'URL `http://localhost:8000/job/1` ou `http://localhost:8000/job/2`

Ajoutons maintenant l'action correspondante dans notre contrôleur :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

use App\Entity\Job;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class JobController extends AbstractController
{
    // ...

    public function show(EntityManagerInterface $em, int $id): Response
    {
        $job = $em->getRepository(Job::class)->find($id);
        if (null === $job) {
            throw new NotFoundHttpException();
        }

        return $this->render('job/show.html.twig', [
            'job' => $job,
        ]);
    }
}
{% endhighlight %}

> La vue affichée par cette action ne sera pas détaillée car elle ne présente pas d'intérêt pour le routing. Je vous invite à récupérer le contenu du fichier directement sur {% ext le dépôt du projet|https://github.com/jdecool/jobeet/tree/05-les-routes/templates/job/show.html.twig %}.

Nous découvrons dans ce contrôleur comment décrire une route contenant un paramètre dynamique. Dans l'exemple précédent, le paramètre `$id` est automatiquement extrait de l'URL et transmis à l'action de notre contrôleur. Symfony faisant correspondre le nom du paramètre de notre configuration avec le nom de la variable de présent dans la définition de notre action.

> Notons au passage l'utilisation de la méthode `find` de Doctrine permettant de récupérer l'objet via sa clé primaire de base de données.

Après avoir effectué notre requête en base de données avec Doctrine, il convient de contrôler que l'identifiant transmis correspond bien à une offre d'emploi. Si ce n'est pas le cas, nous générons une exception afin d'indiquer que l'utilisateur tente d'accéder à une ressource qui n'existe pas.

> La classe `Symfony\Component\HttpKernel\Exception\NotFoundHttpException` est une exception fournie par le composant `HttpFundation` de Symfony. Elle permet de lancer d'indiquer au framework que l'erreur rencontrée est de type "ressource introuvable". Le framework générera ainsi automatiquement une réponse renvoyant le code HTTP 404 au navigateur.

Nous avons créé une route qui permet de capter un paramètre correspondant à l'identifiant d'une offre d'emploi. Mais à ce stade, nous n'avons défini aucune restriction sur ce paramètre. Une donnée de type numérique est attendue, mais actuellement rien n'empêche un utilisateur d'entrer une URL du type `/job/mon-offre`. Cette adresse est valide mais va provoquer une erreur car nous avons indiqué dans notre action que la paramètre `$id` était de type `int`.

Pour éviter ce problème, il est possible de définir des règles de validations des adresses pouvant être captées par nos routes. Ajoutons donc un prérequis sur le paramètre `{id}` afin d'indiquer que ce dernier ne doit prendre en compte que les valeurs numériques entières.

{% highlight yaml %}
# config/routes.yaml

# ...
job_show:
    path: /job/{id}
    defaults: { _controller: 'App\Controller\JobController::show' }
    requirements:
        id: '\d+'

# ...
{% endhighlight %}

Pour définir une contrainte sur un paramètre de notre URL, nous utilisons le mot-clé `requirements` qui permet de définir une liste de prérequis. Dans l'exemple précédent, nous spécifions que le paramètre `id` doit être de valeur numérique.

> Le format des validateurs est en réalité une expression régulière. Le format raccourci `\d+` correspond en réalité à l'expression `[0-9]+`.

Si nous réfléchissons d'un point de vue SEO, nos URL de type `/job/1` ne sont pas très explicites et pourraient pénaliser notre référencement dans les moteurs de recherche. Une URL du type `/job/sensio-labs/1/web-developer` serait plus pertinente car elle décrit mieux la ressource à laquelle elle fait référence. Effectuons cette modification :

{% highlight yaml %}
# config/routes.yaml

# ...
job_show:
    path: /job/{company}/{location}/{id}/{position}
    defaults: { _controller: 'App\Controller\JobController::show' }
    requirements:
        id: '\d+'
        company: '[A-Za-z0-9\-]+'
        location: '[A-Za-z0-9\-]+'
        position: '[A-Za-z0-9\-]+'

# ...
{% endhighlight %}

Et adaptons notre classe `JobController` en conséquence :

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

use App\Entity\Job;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

class JobController extends AbstractController
{
    // ...

    public function show(EntityManagerInterface $em, int $id, string $company, string $position, string $location): Response
    {
        // Faire les contrôles sur les variables $company et $position
        // ou les inclures dans la requête SQL
        $job = $em->getRepository(Job::class)->find($id);
        if (null === $job) {
            throw new NotFoundHttpException();
        }

        return $this->render('job/show.html.twig', [
            'job' => $job,
        ]);
    }
}
{% endhighlight %}

> Nous n'implémenterons pas les requêtes ni les vérifications, ce n'est pas l'objectif de ce tutorial. Qui se concentre sur l'écriture d'une application avec Symfony.

Maintenant que nous avons ajouté notre page permettant de visualiser le détail d'une offre d'emploi, nous allons devoir mettre à jour les liens permettant à un utilisateur de naviguer au sein de notre application. Pour faciliter la navigation entre les pages dans nos templates Twig, le module `TwigBridge` met en place des fonctions permettant de faire référence à une page au travers du nom de la route associée. C'est notamment le cas de la fonction `path`. Cette dernière prend en paramètre le nom de la route à laquelle nous allons faire référence ainsi que les différentes variables nécessaires à la génération de l'URL.

{% highlight html %}{% raw %}
<!-- templates/job/index.html.twig -->
{% extends "base.html.twig" %}

{% block body %}
    <h1 class="my-4">Liste des offres</h1>

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

    <!-- ... -->
{% endblock %}
{% endraw %}{% endhighlight %}

> Pour générer le lien vers une offre d'emploi, nous avons utilisé les propriétés `job.companySlug`, `job.locationSlug` et `job.positionSlug` de notre classe `Job`. Ces dernières correspondent en réalité à des appels de méthode (au besoin, je vous invite à relire le [chapitre concernant Twig]({% post_url 2017-09-29-tutorial-jobeet-symfony-4-partie-4a-le-controleur-et-la-vue %}) pour plus d'informations). L'implémentation des méthodes ne sera pas détaillée, consultez {% ext le code source de la classe|https://github.com/jdecool/jobeet/tree/05-les-routes/src/Entity/Job.php %} pour plus de détails.

Au fur et à mesure que nous allons ajouter des fonctionnalités à notre application, cette dernière va grossir et contenir de plus en plus de code et de configuration. Il peut être parfois utile de lister l'ensemble des routes disponibles sans pour autant devoir parcourir les différents fichiers de configuration. Pour cela, Symfony met à disposition des développeurs une commande permettant d'afficher ces dernières :

{% img cmd-debug-router.png %}

Cette commande permet également d'obtenir des informations détaillées sur une route. Il suffit pour cela d'indiquer le nom de la route en question :

{% img cmd-debug-router-details.png %}

Ce chapitre a introduit la notion de route et explique comment créer des liens entre les pages de votre application. Vous avez ainsi appris à utiliser le composant `Routing` de Symfony. Le prochain sera consacré à l'approfondissement du concept de modèle. Nous y expliquerons plus en détail comment structurer l'information et faire des requêtes complexes.

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}). Le code source de cette application est également disponible sur {% ext Github|https://github.com/jdecool/jobeet/tree/05-les-routes %}. Vous trouvez une branche associée à l'état du projet après chaque chapitre.
