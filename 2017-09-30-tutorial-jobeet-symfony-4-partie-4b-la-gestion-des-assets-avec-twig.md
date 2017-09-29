---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 4B: La gestion des assets avec Twig"
menu: blog
priority: 0.80
type: jobeet
---

Dans la section précédente, nous avons commencé à afficher nos premières pages
et avons défini une charte graphique. Nous avons donc inclus un certain nombre
de fichiers CSS, Javascript et utilisé des images. Voyons avant de continuer
quelques bonnes pratiques sur la gestion des assets.

> Les assets sont des fichiers statiques qui sont utilisés par l'interface de
> notre site. Il peut s'agir de fichiers CSS (pour la mise en page et la charte
> graphique du site), Javascripts (pour gérer l'interactivité de notre interface
> avec l'utilisateur), d'images ou par exemple des fichiers pouvant être 
> téléchargées par l'utilisateur.

Tout d'abord, revoyons notre template `base.html.twig` qui inclut nos différentes
feuilles de style :

{% highlight html %}{% raw %}
# templates/base.html.twig
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

Pour inclure nos différents fichiers, nous avons spécifié le chemin complet
pour accéder à nos ressources. Cette pratique n'est pas recommandée car elle
possède de nombreux désavantages :

* Elle rend nos *templates verbeux* en nous obligeant à écrire pour chaque fichier
le chemin complet pour y accéder. 
* Le *versionnement des fichiers* est compliqué car il nécessite d'être géré manuellement
pour chaque ressource.
* Le *déplacement des assets* peut être source d'erreurs.
* L'*utilisation d'un CDN* est impossible avec cette gestion manuelle.

Afin de résoudre toutes ces problématiques, il existe de nombreux composants PHP
permettant de mettre en place une gestion d'asset simple, efficace et maintenable.
Symfony propose également un composant pour ça, installons-le :

{% highlight bash %}
$ composer require symfony/asset

Using version ^3.3 for symfony/asset
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 1 install, 0 updates, 0 removals
  - Installing symfony/asset (v3.3.9): Downloading (100%)
Writing lock file
Generating autoload files
Executing script make cache-warmup [OK]
Executing script assets:install --symlink --relative public [OK]
{% endhighlight %}

Une fois téléchargé, le composant est tout de suite prêt à être utilisé dans
notre projet. Lors de notre découverte de Twig, nous avons rapidement évoqué le
composant `TwigBridge` comme étant un module permettant d'étendre le fonctionnement
de Twig en ajoutant des fonctionnalités spécifiques aux différents modules de
de Symfony. Le composant Asset en fait partie. `TwigBridge` fournit ainsi une
fonction `asset` permettant d'accéder à une ressource.

Modifions notre template d'origine :

{% highlight html %}{% raw %}
# templates/base.html.twig
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <title>{% block title %}Jobeet{% endblock %}</title>
        <link rel="stylesheet" href="{{ asset('vendor/bootstrap/css/bootstrap.min.css') }}">
        <link rel="stylesheet" href="{{ asset('vendor/1-col-portfolio.css') }}">
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

        <script src="{{ asset('vendor/jquery/jquery-3.2.1.slim.min.js') }}"></script>
        <script src="{{ asset('vendor/popper/popper.min.js') }}"></script>
        <script src="{{ asset('vendor/bootstrap/js/bootstrap.min.js') }}"></script>
        {% block javascripts %}{% endblock %}
    </body>
</html>
{% endraw %}{% endhighlight %}

> Nous pouvons également modifier l'affichage de l'image d'une offre d'emploi
> dans le fichier {% ext `templates/job/index.html.twig`|https://github.com/jdecool/jobeet/blob/04b-assets-twig/templates/job/index.html.twig#L10 %}.
> Notons au passage l'utilisation de l'opérateur de concaténation de chaine de
> caractère Twig `~`.

Peut-être que vous ne remarquez que très peu de différences. Pour le moment, nous
avons juste utilisé notre fonction `asset` mais le code reste le même et le chemin
vers le fichier est toujours présent.

S'il y a peu de différences, c'est tout d'abord parce que nous avons placé nos
fichiers dans le répertoire `public` (qui est le point d'entrée du serveur HTTP
pour rechercher nos fichiers accessibles publiquement). C'est donc le répertoire
où le composant Asset va rechercher nos fichiers par défaut.

Imaginons que nous souhaitons changer le répertoire contenant nos ressources,
pour par exemple le déplacer dans un sous-répertoire `public/assets`. Il ne
sera maintenant plus nécessaire de modifier tous les chemins des fichiers que
nous avons utilisés. Il suffira de modification la configuration du framework
afin de spécifier le répertoire racine qui contient nos assets comme dans
l'exemple ci-dessous :

{% highlight yaml %}
# config/packages/framework.yml
framework:
    # ...
    assets:
        base_path: '/assets'
{% endhighlight %}

Au travers de cet exemple, il est possible de commencer à voir les avantages
d'utiliser un composant pour gérer nos ressources. Une autre des fonctionnalités
majeures du module `Asset` est la gestion du versionnement des fichiers. Pour
plus d'informations sur le versionnement des assets, je vous propose de consulter
la {% ext documentation officielle|http://symfony.com/doc/current/components/asset.html#versioned-assets %}
qui fournira tous les informations utiles.

Si vous souhaitez spécifier une version d'asset dans notre projet, voici la
configuration que vous allez devoir renseigner :

{% highlight yaml %}{% raw %}
# config/packages/framework.yml
# app/config/config.yml
framework:
    # ...
    assets:
        version: 'v2' # asset('/images/logo.png') ==> /images/logo.png?v2
{% endraw %}{% endhighlight %}

> Le versionnement des assets est très utile pour gérer la mise en cache de nos
> ressources. C'est une pratique courante pour optimiser les données devant être
> renvoyées par les serveurs HTTP et économiser des appels réseaux.

Avant de terminer sur la gestion des ressources, évoquons une dernière
fonctionnalité essentielle : la gestion des CDN.

> Un *Content* *Delivery* *Network* (CDN ou réseau de diffusion de contenu en
> français) est une plate-forme de serveurs hautement distribuée optimisée pour
> diffuser du contenu. Les CDN ont l'avantage d’accélérer le chargement des pages.

Pour mettre en place la gestion d'un CDN, il suffit comme pour le versionnement
des ressources, de spécifier les domaines qui vont être utilisés dans le fichier
de configuration. Symfony sélectionnera un domaine (si vous n'en spécifiez pas un
lors de l'utilisaton de la fonction `asset`) à chaque génération d'un page.

{% highlight yaml %}{% raw %}
# config/packages/framework.yml
framework:
    # ...
    assets:
        base_urls:
            - 'http://cdn.example.com/' # asset('/images/logo.png') ==> http://cdn.example.com/images/logo.png
{% endraw %}{% endhighlight %}

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/04b-assets-twig %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
