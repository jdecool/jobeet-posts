---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 4A: Le contrôleur et la vue"
menu: blog
priority: 0.80
type: jobeet
---

Nous avons jusqu'à présent entrevu le fonctionnement de Doctrine en créant une
base de données et en y insérant un jeu de données afin d'avoir des données
initiales, nous évitant ainsi d'avoir une application vide. Nous allons 
maintenant pouvoir commencer à afficher nos premières pages web.

Un projet Symfony repose sur le pattern MVC (*M*odel *V*iew *C*ontroller ou *M*odel
*V*ue *C*ontrôleur en français). Ce type d'architecture permet d'organiser le code
en le séparant en trois couches :

* La couche `modèle` contenant le traitement logique de vos données (on y retrouve
les traitements métier, les accès à la base de données, ...).
* La `vue` est la modélisation de l'IHM (Interface Homme Machine). Elle représente
ce qui est rendu à l'utilisateur (sous la forme d'une page Web, d'une commande
d'un terminal, de données JSON/XML, ...).
* Le `contrôleur` correspond au code faisant le lien entre le `modèle` et la `vue`.
Il récupère les données utilisateurs pour y appliquer les traitements et donner
les résultats à la vue.

Démarrons par ce dernier. Comme nous venons brièvement de le dire, le contrôleur
est la couche qui va exécuter les traitements liés à notre application et
transmettre les résultats à la vue pour que ces derniers puissent être affichés
à l'utilisateur. Dans notre projet Web, cela va se matérialiser par des classes
qui vont contenir des fonctions qui seront appelées en fonction d'une URL. Ces
dernières renverront un objet de type `Symfony\Component\HttpFoundation\Response`
qui est l'abstraction d'une réponse HTTP dans Symfony.

Ecrivons un premier contrôleur que nous allons nommer `DefaultController` et que
nous allons placer dans le répertoire `src/Controller`. Ce dernier, contiendra une
fonction qui permettra d'afficher un message dans le navigateur.

{% highlight php %}
<?php // src/Controller/DefaultController.php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;

class DefaultController
{
    public function index(): Response
    {
        return new Response('Accueil Jobeet - Hello');
    }
}
{% endhighlight %}

Pour tester ce code, nous allons devoir indiquer au framework comment accéder
à ce contrôleur. Pour cela nous allons éditer le fichier `config/routes.yaml`.
Sans rentrer dans les détails (car c'est le sujet du prochain chapitre), ce
fichier permet d'indiquer à Symfony l'URL qui déclenchera l'appel de la méthode
`index` de notre contrôleur.

Pour le moment, décommentons simplement les 3 premières lignes :

{% highlight yaml %}
# config/routes.yaml

index:
    path: /
    defaults: { _controller: 'App\Controller\DefaultController::index' }

# Depends on sensio/framework-extra-bundle, doctrine/annotations, and doctrine/cache
#   install with composer req sensio/framework-extra-bundle annot
#controllers:
#    resource: ../src/Controller/
#    type: annotation
{% endhighlight %}

Nous pouvons maintenant démarrer notre serveur Web via l'exécution de la commande
`make serve` dans un terminal et se rendre à l'adresse {% ext http://localhost:8000|http://localhost:8000 %}
avec son navigateur pour visualiser le résultat.

{% img browser.png %}

Pour pouvoir exécuter les différents traitements de notre application, le contrôleur
doit pouvoir récupérer les données saisies par les utilisateurs. Dans un environnement
Web, ces informations sont transmises par le navigateur au sein d'une requête HTTP.

Symfony utilise un composant {% ext `HttpFoundation`|https://symfony.com/components/HttpFoundation %},
dont l'objectif est de fournir une couche objet permettant de travailler avec le
protocole HTTP. Nous avons, dans les exemples précédents, déjà utilisé un objet
`Response` issue de ce composant. Nous allons maintenant utiliser un objet 
`Symfony\Component\HttpFoundation\Request` nous permettant d'accéder à toutes
les informations d'une requête HTTP.

Pour ce faire, Symfony passe automatiquement en paramètre une instance d'un objet
`Request` aux actions de nos contrôleurs si le paramètre est présent (Symfony 
détecte automatiquement la variable grâce à son typage).

Complétons notre code précédent pour récupérer un paramètre `name` et afficher
le nom de la personne à saluer.

{% highlight php %}
<?php // src/Controller/DefaultController.php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class DefaultController
{
    public function index(Request $request): Response
    {
        return new Response('Accueil Jobeet - Hello '.$request->get('name', 'World'));
    }
}
{% endhighlight %}

> Dans cet exemple, nous utilisons la méthode `Request::get` pour récupérer un
> paramètre de la requête. Ce paramètre peut être transmis via l'URL ou au travers
> d'une requête de type POST. S'il n'est présent dans aucun des cas, nous avons
> choisi ici de retourner la valeur par défaut "World". La valeur par défaut est
> facultative, si rien n'est spécifié et que le paramètre n'est pas présent `NULL`
> sera renvoyé.

Intéressons-nous maintenant à la couche vue. C'est cette dernière qui renvoie et
met en forme les informations qui seront affichées à l'utilisateur. Le format de
retour des données dépend de plusieurs paramètres dont le contexte d'utilisation. 
Dans le cas d'une navigation Web classique, l'application va retourner des données
au format HTML, alors que dans le cas d'une API, les données pourraient être
renvoyées au format JSON, XML ou n'importe quel autre format. Dans ce cas chapitre
nous allons travailler exclusivement avec des pages HTML.

Comme nous l'avons déjà évoqué auparavant, Symfony 4 laisse libre le développeur
de choisir les outils qu'il souhaite utiliser. Le framework ne prend aucun parti
pris. De ce fait et contrairement aux versions précédentes, Symfony n'est plus
fourni avec un moteur de templating par défaut.

> Les moteurs de templating simplifient le travail d'écriture HTML en améliorant 
> la lisibilité du code, son organisation et sa maintenance. Ces derniers sont
> généralement fournis avec un ensemble de fonctions de haut niveau permettant
> entre autres d'afficher des variables PHP, de créer des macros, d'utiliser des
> structures de boucles et de contrôles, etc.

Dans ce tutorial, nous avons fait le choix d'utiliser {% ext Twig|https://twig.symfony.com %}.
Twig a été créé par l'agence SensioLabs, les créateurs du framework Symfony. Ce
moteur de templating était jusqu'à maintenant celui qui était fourni par défaut
avec le framework. Sa simplicité et sa puissance en font le moteur le plus populaire
PHP.

Avant de pouvoir commencer à nous servir de Twig dans notre projet, nous allons
devoir installer les dépendances nécessaires. Pour fonctionner dans notre projet
Symfony, plusieurs composants sont requis :

* `Twig`: le moteur de template en lui-même. Ce dernier n'est pas couplé à Symfony
et peut ainsi être réutilisé dans n'importe quel projet (avec ou sans framework).
* `TwigBridge`: permet d'intégrer des nouvelles fonctionnalités au moteur de
template qui sont liées aux différents composants du framework (formulaires,
internationalisation, ...).
* `TwigBundle`: l'intégration du moteur Twig dans Symfony.

Comme pour toute dépendance PHP, nous allons utiliser Composer (et Symfony Flex)
pour installer Twig. Nous allons ainsi demander l'installation de `symfony/twig-bundle`.
Ce dernier ayant besoin des deux autres composants pour fonctionner, toutes les
dépendances requises au fonctionnement de Twig dans notre projet seront mises en
place.

{% highlight bash %}
$ composer require symfony/twig-bundle

Using version ^3.3 for symfony/twig-bundle
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 3 installs, 0 updates, 0 removals
  - Installing twig/twig (v2.4.3): Loading from cache
  - Installing symfony/twig-bridge (v3.3.9): Downloading (100%)
  - Installing symfony/twig-bundle (v3.3.9): Downloading (100%)
Writing lock file
Generating autoload files
Symfony operations: 1 recipe
  - Configuring symfony/twig-bundle (3.3): From github.com/symfony/recipes:master
Executing script make cache-warmup [OK]
Executing script assets:install --symlink --relative public [OK]
{% endhighlight %}

> Il est également possible d'utiliser une notation raccourcie pour installer Twig
> au travers de la commande `composer require twig`. Cela est possible grace à
> Symfony Flex qui permet de définir des noms alternatifs à des dépendances Composer.
> Dans le chapitre précédent, nous aurions pu utiliser la commande `composer require orm`
> pour installer Doctrine, cette dernière commande étant un alias de la commande
> `composer require orm/pack`.

Maintenant que Twig est installé, nous allons pouvoir créer et afficher notre
premier template. Pour cela, nous allons commencer par injecter le moteur de
templating dans notre contrôleur. Nous pourrons ensuite, utiliser ce dernier depuis
nos actions pour récupérer le rendu de nos templates.

{% highlight php %}
<?php // src/Controller/DefaultController.php

namespace App\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Twig\Environment;

class DefaultController
{
    private $twig;

    public function __construct(Environment $twig)
    {
        $this->twig = $twig;
    }

    public function index(Request $request): Response
    {
        return new Response($this->twig->render('home.html.twig', [
            'name' => $request->get('name', 'World')
        ]));
    }
}
{% endhighlight %}

> Notez qu'il n'est pas nécessaire de faire quoi que ce soit pour injecter notre
> objet `Twig\Environment` dans le constructeur de notre contrôleur. Le composant 
> d'injection de dépendance de Symfony utilise l'{% ext autowiring|https://symfony.com/doc/current/service_container/autowiring.html %}
> pour injecter notre dépendance automatiquement.

Au moment de l'installation de Twig, Flex a préparé un dossier `template` à la
racine de notre projet et a automatiquement configuré Twig pour que ce dernier
aille chercher nos vues dans ce répertoire. Nous allons donc y créer un fichier
`home.html.twig`.

{% highlight bash %}{% raw %}
# templates/home.html.twig
Hello {{ name }}
{% endraw %}{% endhighlight %}

> Pour afficher les variables que nous avons passé à notre template Twig, il faut
> utiliser la syntaxe {% raw %}`{{ variable }}`{% endraw %}.

Pour afficher le contenu de notre template, nous appelons la méthode `render` de
Twig et passons le résultat à notre objet `Response` pour que le résultat puisse
être retourné à l'utilisateur.

Pour afficher le résultat de notre vue, cette syntaxe est un peu longue. Effectivement,
nous allons devoir injecter dans chacun de nos contrôleurs une instance de Twig,
récupérer le contenu d'un template et créer la réponse associée.

Heureusement, Symfony met à notre disposition des outils pour simplifier notre
travail et surtout mutualiser ce code au travers de la classe `AbstractController`.
Cette dernière met à disposition une méthode `render` qui récupérera automatiquement
une instance de Twig et créera notre objet `Response` en un appel de méthode.

Notre contrôleur peut ainsi être réécrit de la manière suivante :

{% highlight php %}
<?php  // src/Controller/DefaultController.php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class DefaultController extends AbstractController
{
    public function index(Request $request): Response
    {
        return $this->render('home.html.twig', [
            'name' => $request->get('name', 'World')
        ]);
    }
}
{% endhighlight %}

Nous connaissons à présent le fonctionnement des contrôleurs et comment ces
derniers communiquent avec la vue pour afficher nos données à l'utilisateur.
Nous allons maintenant pouvoir afficher les premières pages de notre projet
Jobeet. Commençons par la page de listing des offres d'emploi.

Créons pour cela un contrôleur dédié à la gestion des offres et ajouter une
méthode pour récupérer les offres disponibles.

{% highlight php %}
<?php // src/Controller/JobController.php

namespace App\Controller;

use App\Entity\Job;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class JobController extends AbstractController
{
    public function index(EntityManagerInterface $em)
    {
        $jobs = $em->getRepository(Job::class)->findAll();

        return $this->render('job/index.html.twig', [
            'jobs' => $jobs,
        ]);
    }
}
{% endhighlight %}

Nous injectons ici un objet de type `EntityManagerInterface` directement dans
notre méthode grace à l'autowiring (Symfony va détecter et injecter l'instance
de l'objet qui implémente l'interface).

> L'autowiring fonctionne avec une interface uniquement s'il n'existe qu'une seule
> classe qui implémente l'interface en question. Si deux classes implémentent la
> même interface, Symfony ne sera pas capable de savoir quelle instance injecter.

L'`EntityManager` est l'objet Doctrine qui permet de manipuler nos entités. Dans
notre code, nous demandons une instance du repository (c'est-à-dire la classe qui
permet de faire les requêtes en base de données) de notre entité `Job` pour ensuite
récupérer la liste des emplois disponibles.

> Doctrine fournit un ensemble de méthodes par défaut permettant de récupérer nos
> objets persistés. Signalons entre autres les fonctions {% ext `find`|https://goo.gl/EyWW5N %},
> {% ext `findBy`|https://goo.gl/EyWW5N %}, `findOneBy` et `findAll`.

Ajoutons la vue qui va afficher nos résultats :

{% highlight html %}{% raw %}
<!-- templates/job/index.html.twig -->

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Jobeet - Liste des jobs</title>
        <link rel="icon" type="image/x-icon" href="{{ asset('favicon.ico') }}" />
    </head>
    <body>
        <ul>
            {% for job in jobs %}
                <li>{{ job.position }}</li>
            {% endfor %}
        </ul>
    </body>
</html>
{% endraw %}{% endhighlight %}

> Pour afficher la page, il sera nécessaire de modifier le fichier de configuration
> {% ext `config/routes.yaml`|https://github.com/jdecool/jobeet/blob/04-controleur-vue/config/routes.yaml %}
> en modifiant le contrôleur devant être appelé.

L'exemple ci-dessus nous permet de découvrir de nouveaux éléments du langage de
Twig. Tout d'abord, les instructions d'itérations qui nous permettent de parcourir
des données de type `array` ou plus généralement des données {% ext `iterable`|http://php.net/manual/en/language.types.iterable.php %}
de PHP. Il s'agit d'une boucle {% ext `for`|https://twig.symfony.com/doc/2.x/tags/for.html %}
permettant de parcourir une liste d'éléments.

Nous constatons également qu'il est possible de passer à notre template Twig des
instances d'objets et d'accéder aux propriétés de ces derniers avec l'opérateur `.`
(cela fonctionnement également pour accéder à des tableaux).

> Bien que la propriété `position` de notre objet `Job` soit privée, Twig parvient
> à afficher cette dernière. Twig possède un mécanisme qui va automatiquement
> trouver la méthode `get` associé à la propriété si cette dernière n'est pas
> accessible directement (si elle n'est pas `public`). Il est toutefois possible
> d'appeler directement la méthode `getPosition()` ou n'importe quelle autre
> méthode depuis notre template.

Pour le moment nous avons copié la totalité de la structure HTML dans notre fichier.
Pour éviter d'avoir à dupliquer de nombreuses lignes de code dans tous nos templates,
nous allons mutualiser le code qui va être commun à toutes les pages.

Si vous jetez un oeil aux fichiers qui ont été ajoutés dans le dossier `templates`
par Flex lors de l'installation de Twig, vous noterez la présence d'un fichier
`base.html.twig`.

{% highlight html %}{% raw %}
<!-- templates/base.html.twig -->

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>{% block title %}Jobeet{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>
{% endraw %}{% endhighlight %}

Ce fichier correspond à un template qui a vocation de définir la mise en page
globale de notre application. Twig fonctionne sur le principe d'héritage. Cela
signifie que vous allez pouvoir définir des templates qui contiendront des éléments
qui pourront ensuite être surchargés dans d'autres templates. Pour cela, Twig
utilise un système de blocs ({% ext `block`|https://twig.symfony.com/doc/2.x/tags/block.html %}).

Utilisons le fichier `templates/base.html.twig` pour définir la mise en page de
Jobeet. Pour commencer, nous allons étendre ce dernier dans notre template
`templates/job/index.html.twig` et surcharger le bloc `body` pour y faire apparaître
notre contenu :

{% highlight html %}{% raw %}
<!-- templates/job/index.html.twig -->
{% extends "base.html" %}

{% block body %}
    <ul>
        {% for job in jobs %}
            <li>{{ job.position }}</li>
        {% endfor %}
    </ul>
{% endblock %}
{% endraw %}{% endhighlight %}

Si vous rechargez la page, le résultat devrait être identique, mais nous avons
beaucoup moins de code dans notre template, qui se concentre maintenant uniquement
sur une tâche bien précise : afficher notre liste d'offres d'emploi.

Pour que nous interfaces soient un plus agréables à utiliser, nous allons y ajouter
un peu de style avec du CSS. Pour gagner du temps, nous allons utiliser le framework
{% ext boostrap|http://getbootstrap.com/ %} au travers de ce {% ext thème|https://startbootstrap.com/template-overviews/1-col-portfolio/ %}.

Une fois les fichiers téléchargés, nous allons placer les différents fichiers CSS
et Javascript dans le dossier `public` (car ces derniers doivent être desservis
par le serveur HTTP). Nous les organiserons dans un dossier `vendor` comme vous
pouvez le voir dans le {% ext dépôt Github|https://github.com/jdecool/jobeet/tree/04-controleur-vue/public/vendor %}.

Commençons ensuite par modifier notre mise en page globale :

{% highlight html %}{% raw %}
<!-- templates/base.html.twig -->

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <title>{% block title %}Jobeet{% endblock %}</title>
        <link rel="stylesheet" href="/vendor/bootstrap/css/bootstrap.min.css">
        <link rel="stylesheet" href="/vendor/1-col-portfolio.css">
        {% block stylesheets %}{% endblock %}
    </head>
    <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-dark fixed-top">
            <div class="container">
                <a class="navbar-brand" href="#">Jobeet</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarResponsive" aria-controls="navbarResponsive" aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
            </div>
        </nav>

        <div class="container">
            {% block body %}{% endblock %}
        </div>

        <script src="/vendor/jquery/jquery-3.2.1.slim.min.js"></script>
        <script src="/vendor/popper/popper.min.js"></script>
        <script src="/vendor/bootstrap/js/bootstrap.min.js"></script>
        {% block javascripts %}{% endblock %}
    </body>
</html>
{% endraw %}{% endhighlight %}

Nous conservons les blocs Twig `stylesheets` et `javascripts` même si nous ne les
utiliserons pas pour le moment. C'est dernier seront utiles à l'avenir pour inclure
des fichiers CSS ou Javascript spécifiques à certaines vues.

Pour finir, mettons en forme la présentation de nos offres d'emploi :

{% highlight html %}{% raw %}
<!-- templates/job/index.html.twig -->
{% extends "base.html.twig" %}

{% block body %}
    <h1 class="my-4">Liste des offres</h1>

    {% for job in jobs %}
        <div class="row">
            <div class="col-md-7">
                <a href="#">
                    <img class="img-fluid rounded mb-3 mb-md-0" src="/images/{{ job.logo }}" alt="{{ job.company }}">
                </a>
            </div>
            <div class="col-md-5">
                <h3>{{ job.position }}</h3>
                <p>{{ job.description }}</p>
                <p>Posted on {{ job.createdAt|date("m/d/Y") }}</p>
                <a class="btn btn-primary" href="#">See more</a>
            </div>
        </div>

        <hr>
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

> Les images liées à notre jeu de données sont disponibles directement dans les
> {% ext sources de ce billet|https://github.com/jdecool/jobeet/tree/04-controleur-vue/public/images %}.

Et voilà le rendu final de notre application Jobeet :

{% imgFull final-render.png %}

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/04-controleur-vue %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
