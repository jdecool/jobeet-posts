---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 1: Démarrage du projet"
menu: blog
priority: 0.80
type: jobeet
github: "https://github.com/jdecool/jobeet-posts"
---

Est-il encore nécessaire de présenter Symfony qui est l'un des frameworks PHP
les plus populaires ? Dans cette première partie, nous allons créer la structure
de notre application et voir comment s'organise un projet Symfony.

Démarrons tout d'abord avec les prérequis nécessaires à la mise en place de notre
projet. Concernant la version de PHP que vous allez devoir utiliser, un PHP 7.1.3
sera au minimum nécessaire. Dans cette série de billets, nous partirons du principe
que vous avez installé les extensions permettant d'accéder à une base de données
telle que MySQL ou PostgreSQL (via l'extension PDO et les drivers associés). Le
module XML de PHP (alias `php-xml`) est également requis. Pour finir, nous
supposerons également, que {% ext Composer|https://getcomposer.org %}, l'outil de
gestion des dépendances PHP est installé sur votre machine.

Jusqu'à la version 4 de Symfony, la création d'un nouveau projet se faisait au
travers d'un outil d'installation spécifique nommé {% ext Symfony installer|https://github.com/symfony/symfony-installer %}.
Ce dernier outil est maintenant remplacé par Composer qui possède une commande
{% ext create-project|https://getcomposer.org/doc/03-cli.md#create-project %}
permettant de créer un nouveau projet PHP. Cette commande est ainsi indépendante
du framework. Les équipes de Symfony ont souhaité standardiser la manière de démarrer
un projet PHP sans devoir y ajouter des outils spécifiques. La création de notre
projet Jobeet se fera ainsi au travers de la commande :
`composer create-project symfony/skeleton jobeet`.

Composer va alors créer un dossier pour notre projet en se basant sur le
{% ext modèle de projet Symfony|https://github.com/symfony/skeleton %} et télécharger
les dépendances associées. Dans sa nouvelle version, le framework fournit dorénavant
un composant nommé {% ext Flex|https://github.com/symfony/flex %}, une surcouche
à Composer et qui permet entre autres de configurer automatiquement les dépendances
que vous installez dans votre application.

Analysons la structure de notre projet vide :

{% highlight bash %}
jobeet
├── .env
├── .env.dist
├── Makefile
├── composer.json
├── composer.lock
├── config
│   ├── bundles.php
│   ├── packages
│   │   ├── dev
│   │   │   └── routing.yaml
│   │   ├── framework.yaml
│   │   ├── routing.yaml
│   │   └── test
│   │       └── framework.yaml
│   ├── routes.yaml
│   └── services.yaml
├── public
│   └── index.php
├── src
│   ├── Controller
│   └── Kernel.php
└── vendor
{% endhighlight %}

Contrairement aux versions précédentes, le framework a changé l'arborescence
de ces fichiers afin de se rapprocher d'une arborescence de fichiers que l'on
pourrait retrouver sur des systèmes Unix. On retrouve ainsi les éléments
ci-dessous :

<div class="row">
  <div class="col-2"><code>config</code></div>
  <div class="col-10">Dossier contenant les fichiers de configuration de notre projet</div>
</div>

<div class="row">
  <div class="col-2"><code>public</code></div>
  <div class="col-10">Le répertoire contenant les fichiers désservis par les serveurs HTTP</div>
</div>

<div class="row">
  <div class="col-2"><code>src</code></div>
  <div class="col-10">Accueillera les fichiers sources PHP contenant toute la logique de notre projet</div>
</div>

<div class="row">
  <div class="col-2"><code>vendor</code></div>
  <div class="col-10">Le dossier utilisé par Composer et contenant les dépendances (décrites dans le fichier <code>composer.json</code>) nécessaires au fonctionnement de notre projet</div>
</div>

<div class="row">
  <div class="col-2"><code>.env</code></div>
  <div class="col-10">Fichier contenant la configuration de l'environnement d'exécution de notre code</div>
</div>

<div class="row">
  <div class="col-2"><code>Makefile</code></div>
  <div class="col-10">Décrit les tâches pouvant être exécutées par le framework</div>
</div>

Même si nous n'avons encore aucune ligne de code ni même de configuration (en
réalité Flex s'en est chargé pour nous), nous pouvons démarrer notre application
Symfony Il est possible de démarrer un serveur HTTP au travers de la commande
`make serve` (si vous ne connaissez pas l'outil {% ext Make|https://www.gnu.org/software/make/ %},
je vous recommande vivement de vous renseigner sur ce dernier).

{% img jobeet-serve.png %}

Par défaut, le framework utilise le serveur Web embarqué avec PHP (ce qui peut
être suffisant pour des besoins de développement). Symfony fournit également un
bundle `symfonyweb-server-bundle` (nous reviendrons plus tard sur la notion de
bundle) permettant d'apporter une meilleure expérience développeur pour manipuler
ce dernier.

Une fois le serveur démarré, nous pouvons nous connecter sur notre projet en ouvrant
un navigateur et en tapant l'URL <a href="http://localhost:8000" target="_blank" rel="nofollow">localhost:8000</a>.
Bien entendu, comme nous n'avons encore rien fait, une page d'erreur sera affichée.

{% imgFull jobeet-homepage.png %}

Attardons-nous maintenant le contenu du fichier `.env`. Si vous regardez le contenu
de ce dernier, vous constaterez qu'il contient un certain nombre de paramètres :

{% highlight bash %}
# This file is a "template" of which env vars needs to be defined in your configuration or in a .env file
# Set variables here that may be different on each deployment target of the app, e.g. development, staging, production.
# https://symfony.com/doc/current/best_practices/configuration.html#infrastructure-related-configuration

###> symfony/framework-bundle ###
APP_ENV=dev
APP_DEBUG=1
APP_SECRET=f78d2a48cbd00d92acf418a47a0a5c3e
###< symfony/framework-bundle ###
{% endhighlight %}

Notons tout d'abord les commentaires `###> symfony/framework-bundle ###` et
`###< symfony/framework-bundle ###` présent dans le fichier. Ces derniers
servent de balise de début et de fin à Flex pour ajouter et supprimer automatiquement
la configuration de nos dépendances lors de leurs ajouts ou suppressions dans
notre projet.

Viennent ensuite les différentes variables d'environnement d'exécution de notre
application :

* La variable `APP_ENV` permet de définir l'environnement courant d'exécution du
projet. Généralement 3 environnements sont utilisés : `prod` (l'environnement où
interagissent les utilisateurs finaux), `dev` (l'environnement utilisé pendant
les développements) et `test` (l'environnement utilisé pour tester automatiquement
notre projet).

* La variable `APP_DEBUG` de Symfony fournit un ensemble d'outils permettant de
faciliter la résolution des bugs pouvant survenir lors du développement. Par
exemple, c'est cette dernière qui génère l'affichage de la pile d'appel des fonctions
du framework lorsqu'une erreur se produit (comme lorsque nous avons tenté d'accéder
à l'application via le navigateur).

* Un des principaux avantages d'avoir recours à un framework est d'utiliser un
ensemble de composants réutilisables, régulièrement mis à jour et utilisant des
bonnes pratiques de développement. Cela permet au framework de guider le développeur
et de minimiser les erreurs pouvant faire apparaître des failles de sécurité. La
variable `APP_SECRET` est ainsi utilisée par certains composants en tant que clé
secrète pour générer des jetons de sécurité (comme par exemple lors de la mise en
place de formulaire avec les jetons {% ext CSRF|https://fr.wikidia.org/wiki/Cross-Site_Request_Forgery %}).

C'est sur ce point que nous allons conclure ce premier chapitre. A ce stade, nous
avons créé notre projet Symfony et rapidement fait le tour de l'organisation des
fichiers de notre application. Notre environnement est maintenant prêt et nous
allons prochainement pouvoir commencer à développer.

La section suivante dévoilera les fonctionnalités de notre application et
détaillera les besoins fonctionnels et techniques à satisfaire.

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/01-demarrage-projet %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
