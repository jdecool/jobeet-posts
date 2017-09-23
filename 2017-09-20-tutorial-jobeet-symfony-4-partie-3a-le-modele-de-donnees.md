---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 3A: Le modèle de données"
menu: blog
priority: 0.80
type: jobeet
---

Maintenant que l'aspect fonctionnel de notre projet a été défini, nous allons
pouvoir créer le modèle de données associé à notre application, c'est-à-dire
les classes qui permettront d'interagir avec la base de données. Nous allons
pour cela avoir recours à un ORM. Ce sera également l'occasion de voir comment
ajouter et configurer un module tier dans notre projet.

D'après [les scénarios]({% post_url 2017-09-19-tutorial-jobeet-symfony-4-partie-2-le-projet %}) 
que nous avons précédemment écrits, voici le schéma correspondant aux relations
entre entités que l'on peut en déduire :

{% img database.png %}

En plus des informations, nous avons également ajouté un champ `created_at` et
`updated_at` à certaines tables afin de conserver une trace des dernières
modifications des données que nous allons traiter.

Pour stocker les informations de l'application, nous allons utiliser une base
de données relationnelle. Symfony étant un framework orienté-objet, nous allons
donc manipuler les informations sous la forme d'objet. Les informations de notre
base de données doivent ainsi être mappées avec notre modèle objet et pour cela
nous allons utiliser un ORM.

Symfony 4 laisse le choix au développeur sur les outils qu'il souhaite utiliser.
Contrairement aux versions précédentes, Symfony est livré "vide" et n'impose aucun
choix par défaut. Pour ce tutorial, nous allons faire le choix d'utiliser l'ORM
{% ext Doctrine|http://www.doctrine-project.org/projects/orm.html %} qui est l'ORM
le plus répandu dans l'écosystème PHP. Mais il serait tout à fait possible d'utiliser
une simple connexion {% ext PDO|http://php.net/manual/fr/book.pdo.php %} ou
{% ext Doctrine DBAL|http://www.doctrine-project.org/projects/dbal.html %}, ou un
autre ORM tel que {% ext Propel|http://propelorm.org/ %}, {% ext Eloquent|https://laravel.com/docs/5.0/eloquent %},
{% ext Pomm|http://www.pomm-project.org/ %} ou n'importe quel autre outil.

Avant de créer et configurer les classes de notre modèle de données, nous allons
commencer par télécharger Doctrine. Pour intégrer ce dernier dans notre projet
Symfony, nous devrons récupérer deux dépendances. La première, `doctrine/orm`
est l'ORM en tant que tel. La seconde dépendance consiste à intégrer l'ORM dans
notre architecture Symfony. Symfony facilite le développement et la réutilisation
de module que l'on peut partager entre plusieurs projets. Ces modules (des plugins
en quelque sorte) sont appelés **bundles** dans l'écosystème du framework. Il
convient donc de télécharger la dépendance `doctrine/doctrine-bundle` qui va
intégrer l'ORM dans notre environnement Symfony.

Afin d'éviter au développeur d'avoir à installer deux dépendances distinctes,
les équipes de développement fournissent un méta-paquet Composer permettant
d'installer les deux éléments d'un coup. Pour intégrer Doctrine à notre projet,
il nous suffit donc de récupérer la dépendance `symfony/orm-pack`.

{% highlight bash %}
$ composer require symfony/orm-pack

Using version ^1.0 for symfony/orm-pack
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 13 installs, 0 updates, 0 removals
  - Installing doctrine/inflector (v1.2.0): Loading from cache
  - Installing doctrine/lexer (v1.0.1): Loading from cache
  - Installing doctrine/collections (v1.5.0): Loading from cache
  - Installing doctrine/annotations (v1.5.0): Loading from cache
  - Installing doctrine/common (v2.8.1): Loading from cache
  - Installing symfony/doctrine-bridge (v3.3.8): Loading from cache
  - Installing doctrine/doctrine-cache-bundle (1.3.0): Loading from cache
  - Installing jdorn/sql-formatter (v1.2.17): Loading from cache
  - Installing doctrine/dbal (v2.6.2): Loading from cache
  - Installing doctrine/doctrine-bundle (1.7.0): Loading from cache
  - Installing doctrine/instantiator (1.0.5): Loading from cache
  - Installing doctrine/orm (v2.5.10): Loading from cache
Writing lock file
Generating autoload files
Symfony operations: 2 recipes
  - Configuring doctrine/doctrine-cache-bundle (1.3.0): From auto-generated recipe
  - Configuring doctrine/doctrine-bundle (1.6): From github.com/symfony/recipes:master
Executing script make cache-warmup [OK]
Executing script assets:install --symlink --relative public [OK]
{% endhighlight %}

Pour utiliser un bundle dans un projet, il est nécessaire d'activer ce dernier
afin qu'il soit reconnu par le framework. Cette configuration s'effectue dans le
fichier `config/bundles.php`. Ce fichier retourne un tableau associatif où la clé
des éléments correspond au namespace complet du bundle à activer et dont la
valeur est également un tableau associatif indiquant les environnements pour
lesquels le bundle est actif (ou non).

{% highlight php %}
<?php // config/bundles.php

return [
    'Symfony\Bundle\FrameworkBundle\FrameworkBundle' => ['all' => true],
    'Doctrine\Bundle\DoctrineCacheBundle\DoctrineCacheBundle' => ['all' => true],
    'Doctrine\Bundle\DoctrineBundle\DoctrineBundle' => ['all' => true],
];
{% endhighlight %}

En réalité, l'activation d'un bundle se fait rarement manuellement. Effectivement,
{% ext Symfony Flex|https://github.com/symfony/flex %}, que nous avons évoqué
brièvement lors de la mise en place du projet se chargera de cette action. Il
sera néanmoins parfois nécessaire de corriger la configuration par défaut en
activant ou désactivant le bundle pour certains environnements.

Maintenant que Doctrine est installé et activé dans notre projet, nous allons
pouvoir commencer à paramétrer notre application pour qu'elle puisse accéder à
une base de données. Pour cela nous allons renseigner les paramètres nécessaires
à l'établissement de la connexion. Cette dernière étant propre à l'environnement
d'exécution de notre code, nous allons définir les paramètres dans le fichier
`.env`. Ce dernier a d'ailleurs été prérempli avec une configuration de base par
Flex :

{% highlight bash %}
# .env
# ...

###> doctrine/doctrine-bundle ###
# Format described at http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html#connecting-using-a-url
# For an SQLite database, use: "sqlite:///%kernel.project_dir%/var/data.db"
# Set "serverVersion" to your server version to avoid edge-case exceptions and extra database calls
DATABASE_URL="mysql://root@127.0.0.1:3306/symfony?charset=utf8mb4&serverVersion=5.7"
###< doctrine/doctrine-bundle ###
{% endhighlight %}

> Vous avez certainement noté la présence des fichiers `.env` et `.env.dist`. Le
> fichier `.env` est le fichier qui contient réellement la configuration de notre
> application. Contenant des informations pouvant être très sensibles (comme par
> exemple des mots de passe), il n'est pas versionné.

> C'est pour cela qu'un fichier `.env.dist` est présent. Ce dernier qui lui est
> versionné sert de modèle pour que les développeurs qui vont être ammenés à
> travailler sur le projet puisse connaître les informations à renseigner.

L'exemple de configuration qui a été inséré par Symfony Flex permet de se
connecter à une base de données MySQL. Pour les besoins de ce tutorial, ainsi
que pour éviter l'installation d'un serveur, nous allons utiliser SQLite qui est
un système de base de données ne nécessitant pas une architecture client-serveur.
Il sera nécessaire de vérifier que le driver SQLite de PHP (`php-sqlite3`) soit
installé et activé. Nous allons ensuite modifier la variable d'environnement
`DATABASE_URL` comme suit :

{% highlight bash %}
DATABASE_URL="sqlite:///var/jobeet.db"
{% endhighlight %}

> Notons que l'exemple de configuration pour SQLite fait référénce à un paramètre
> `kernel.project_dir` définit par le framework (reconnaissable parce qu'elle est
> entourée du caractère `%`). Cette variable est censée faire référence à la racine
> de notre dossier de code. Mais cette dernière ne semble pas prise en compte lors
> de l'écriture de ce billet et nous avons donc utilisé un chemin relatif.

> *Mise à jour du 23/09/2017 :* En parcourant le projet sur Github, j'ai trouvé
> l'{% ext issue concernant ce problème|https://github.com/symfony/flex/issues/129 %}
> ainsi que {% ext sa résolution|https://github.com/symfony/symfony/pull/23901 %}.
> Tout devrait rentrer dans l'ordre lors de la publication de la branche 3.4 du
> projet.

Le dossier `var` qui a été créé lors de la mise en place de Doctrine est, par
convention, un dossier qui va contenir les fichiers écrits par notre application
durant son fonctionnement (tel que des logs, des fichiers de cache, ...). C'est
donc tout naturellement dans ce dernier que nous allons stocker le fichier qui
contiendra nos données SQLite.

Nous allons maintenant pouvoir démarrer l'écriture des classes qui vont modéliser
nos données. Par défaut Doctrine est configuré pour travailler avec des annotations.
Personnellement, je préfère séparer le code de sa configuration, c'est pour cela
quand dans ce tutorial, nous utiliserons une configuration en YAML (il est 
également possible d'avoir une configuration en XML). Pour cela, nous allons
éditer le fichier de configuration `config/packages/doctrine.yaml` pour y mettre le
contenu ci-dessous :

{% highlight bash %}
# config/packages/doctrine.yml
doctrine:
    dbal:
        url: '%env(DATABASE_URL)%'
    orm:
        auto_generate_proxy_classes: '%kernel.debug%'
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true
        mappings:
            App:
                is_bundle: false
                type: yml # annotation ou xml
                dir: '%kernel.project_dir%/config/doctrine/mapping' # configuration du mapping
                prefix: 'App\Entity\'
                alias: App
{% endhighlight %}

On retrouve dans cette configuration la variable `%kernel.project_dir%` faisant
référence au dossier racine de notre projet. Lorsqu'un paramètre de configuration
est définit dans le framework. De la même façon, il est possible d'accéder à une
variable d'environnement via la syntaxe `%env(MA_VARIABLE)%` (comme dans le fichier
Doctrine pour accéder à la chaine de connexion à la base de données).

> Si vous analysez l'arborescence des fichiers, vous constaterez qu'il existe
> deux fichiers de configuration Doctrine : `config/packages/doctrine.yaml`
> et `config/packages/prod/doctrine.yaml`.

> Le premier est un fichier de configuration commun à tous les environnements.
> Il est ensuite possible de définir une configuration spécifique pour un 
> environnement (défini par la variable `APP_ENV` du fichier `.env`). Pour 
> cela, il suffit de déposer le fichier de configuration dans un sous-dossier 
> portant le nom de l'environnement pour lequel on souhaite surcharger la 
> configuration et le framework le prendra automatiquement en compte.

Nous n'irons pas plus loin dans la configuration de Doctrine. Si vous après ce
tutorial, vous souhaitez en savoir plus, je vous conseille de consulter la 
{% ext documentation officielle|http://symfony.com/doc/current/doctrine.html %}.

Créons maintenant les classes associées à notre modèle de données. Elles vont
représenter les informations de la base de données sous la forme d'objets PHP
(ces derniers sont appelés des entités) que l'on va pouvoir manipuler dans notre
code. Lors de l'installation de Doctrine, Flex a ajouté un dossier `src/Entity`
dans lequel nous allons créer nos classes.

Les entités sont de simples objets PHP dont les propriétés vont correspondre aux
champs de notre base de données. Commençons par la table la plus simple, la table
`category` :

{% highlight php %}
<?php // src/Entity/Category.php

namespace App\Entity;

class Category
{
    private $id;
    private $name;
}
{% endhighlight %}

Maintenant passons à la table `job`. Un emploi étant lié à une catégorie, notre
table contient une clé étrangère vers la catégorie associée. Dans notre entité,
cette information va se modéliser sous la forme d'une propriété dont la valeur
sera une instance de l'entité associée à la table catégorie (donc un objet
`Category`).

{% highlight php %}
<?php // src/Entity/Job.php

namespace App\Entity;

class Job
{
    private $id;
    private $category; // instance de Category
    private $type;
    private $company;
    private $logo;
    private $url;
    private $position;
    private $location;
    private $description;
    private $howToApply;
    private $token;
    private $isPublic;
    private $isActivated;
    private $email;
    private $expiresAt;
    private $createdAt;
    private $updatedAt;
}
{% endhighlight %}

La table qui va gérer les informations d'affiliation est un peu plus complexe.
Dans notre modèle, une société peut-être être affiliée à plusieurs catégories
et une catégorie peut avoir des affiliations de plusieurs sociétés. Avec une
base de données relationnelle, cela se traduit par la création d'une table
d'association pour gérer cette information (il s'agit de la table `CategoryAffiliate`).

Puisque nous avons dit qu'une table de notre base de données correspondait à un
objet PHP, il devrait donc être nécessaire de créer deux nouveaux objets pour
gérer cette relation. Mais en réalité cette table ne sert qu'à modéliser le fait
qu'un objet `Affialite` est rattaché à plusieurs objets `Category` et vice-versa.
Donc d'un point de vue programmation, un objet `Affiliate` devrait avoir une 
propriété `$categories` qui correspond à un tableau d'objet `Affiliate` et l'entité
`Category` une propriété `$affiliates` correspondant un tableau d'objet `Affiliate`.

Notre ORM est tout à fait capable de gérer cette problématique. Nous allons donc
créer notre objet `Affiliate` avec une propriété `$categories` que nous ferons
correspondre à un tableau d'objet `Category`. Doctrine gérera de manière automatique
et transparente notre table d'association.

> S'il avait été nécessaire de gérer des informations additionnelles, telles
> que par exemple la date de création de l'affiliation ou l'utilisateur ayant
> créé l'affiliation, il aurait été nécessaire de créer une entité supplémentaire
> et de gérer la relation manuellement.

{% highlight php %}
<?php // src/Entity/Affiliate.php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;

class Affiliate
{
    private $id;
    private $categories; // tableau d'objet Category
    private $url;
    private $email;
    private $token;
    private $isActive;
    private $createdAt;

    public function __construct()
    {
        $this->categories = new ArrayCollection();
    }
}
{% endhighlight %}

> Lors de la mise en place d'une relation où l'on va gérer un tableau d'objet,
> il est nécessaire d'initialiser la propriété en question avec une collection
> vide. Pour Doctrine, cela passe par la création d'un objet `ArrayCollection`
> comme dans l'exemple précédent.

N'oublions pas de modifier notre objet `Category` pour y ajouter la propriété
correspondant à nos objets `Affialite`. Une catégorie étant également associée
à plusieurs emplois, nous allons en profiter pour y ajouter la propriété 
correspondante.

{% highlight php %}
<?php // src/Entity/Category.php

namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;

class Category
{
    private $id;
    private $name;
    private $jobs; // tableau d'objet Job
    private $affiliates; // tableau d'objet Affiliate

    public function __construct()
    {
        $this->jobs = new ArrayCollection();
        $this->affiliates = new ArrayCollection();
    }
}
{% endhighlight %}

Il est maintenant temps d'indiquer à Doctrine comment l'ORM va pouvoir faire le
lien entre nos entités et les tables de la base de données. Pour cela, et comme
nous l'avons spécifié précédement, nous allons placer des fichiers de configuration
dans le dossier `config/doctrine/mapping`. Tout comme pour l'écriture des classes,
nous allons créer un fichier de configuration par entité en suivant la convention
`NomDeLaClasse.orm.yml`.

Les fichiers de configuration vont permettre d'indiquer à quelle table correspondent
une entité et les différentes caractéristiques de nos propriétés (colonne de
rattachement, type de données, contraintes d'intégrité, ....).

Commençons par la configuration de notre entité `Category` :

{% highlight yaml %}
# config/doctrine/Category.orm.yml
App\Entity\Category:
    type: entity

    # clé(s) primaire(s)
    id:
        id:
            type: integer
            generator:
                strategy: AUTO

    # colonne(s) de la table
    fields:
        name:
            type: string
            length: 63

    # relation de type un vers plusieurs
    oneToMany:
        jobs:
            targetEntity: Job
            mappedBy: category

    # relation de type plusieurs vers plusieurs
    manyToMany:
        affiliates:
            targetEntity: Affiliate
            inversedBy: categories
{% endhighlight %}

> Comme vous pouvez le constater, les propriétés de correspondant à des relations
> sont dissociés du reste des propriétés. On distingue quatre types de relation,
> {% ext `One-To-Many`|https://goo.gl/ExSdg4 %} (relation de type un vers plusieurs),
> {% ext `Many-To-One`|https://goo.gl/tgffTs %} (plusieurs vers un),
> {% ext `Many-To-Many`|https://goo.gl/WBDHLm %} (relation de plusieurs à plusieurs) et 
> {% ext `One-To-One`|https://goo.gl/NA7LFn %}.

Voici la configuration de l'entité `Job` :

{% highlight yaml %}
# config/doctrine/Job.orm.yml
App\Entity\Job:
    type: entity

    id:
        id:
            type: integer
            generator:
                strategy: AUTO

    fields:
        type:
            type: string
            length: 255
            nullable: true

        company:
            type: string
            length: 255

        logo:
            type: string
            length: 255
            nullable: true

        url:
            type: string
            length: 255
            nullable: true

        position:
            type: string
            length: 255

        location:
            type: string
            length: 255

        description:
            type: text

        howToApply:
            type: text

        token:
            type: string
            length: 255
            unique: true

        isPublic:
            type: boolean
            default: false

        isActivated:
            type: boolean
            default: true

        email:
            type: string
            length: 255

        expiresAt:
            type: datetime

        createdAt:
            type: datetime

        updatedAt:
            type: datetime
            nullable: true

    manyToOne:
        category:
            targetEntity: Category
            inversedBy: jobs
            joinColumn:
                name: category_id
                referencedColumnName: id
{% endhighlight %}

Et pour finir le mapping correspondant à l'entité `Affiliate` :

{% highlight yaml %}
# config/doctrine/Affiliate.orm.yml
App\Entity\Affiliate:
    type: entity

    id:
        id:
            type: integer
            generator:
                strategy: AUTO

    fields:
        url:
            type: string
            length: 255

        email:
            type: string
            length: 255
            unique: true

        token:
            type: string
            length: 255
            unique: true

        createdAt:
            type: datetime

    manyToMany:
        categories:
            targetEntity: Category
            mappedBy: affiliates
{% endhighlight %}

Lorsque nous allons enregistrer un emploi ou une affiliation, nous souhaiterions
connaître la date de création et/ou de modification de la donnée écrite. Les
entités correspondantes possèdent une propriété `createdAt` et/ou `updatedAt`.
Plutôt que de devoir gérer manuellement cette information, nous allons déléguer
ce travail à Doctrine.

En effet l'ORM possède un gestionnaire d'événement sur lequel nous allons nous
brancher afin d'être notifié lors de l'enregistrement et la modification d'une
entité. Nous allons donc ajouter cette configuration au mapping de nos entités
afin d'indiquer la méthode de l'objet qui sera appelée lors de la propagation
de l'événement.

Pour l'entité `Job` :

{% highlight yaml %}
# config/doctrine/Job.orm.yml
App\Entity\Job:
    # ...

    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue ] # appelé lors de la création de l'entité
        preUpdate:  [ setUpdatedAtValue ] # appelé lors de la modification de l'entité
{% endhighlight %}

Pour l'entité `Affiliate` :

{% highlight yaml %}
# config/doctrine/Affiliate.orm.yml
App\Entity\Affiliate:
    # ...

    lifecycleCallbacks:
        prePersist: [ setCreatedAtValue ]

{% endhighlight %}

Il ne faudra pas oublier d'ajouter les méthodes correspondantes dans les classes
associées. Ces dernières sont appelées avec un paramètre de type `LifecycleEventArgs`
contenant un certain nombre d'informations sur le contexte d'exécution de l'ORM.

{% highlight php %}
// src/Entity/Job.php
// ...

use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

class Job
{
    // ...

    public function setCreatedAtValue(LifecycleEventArgs $event)
    {
        $this->createdAt = new DateTime();
    }

    public function setUpdatedAtValue(LifecycleEventArgs $event)
    {
        $this->updatedAt = new DateTime();
    }
}
{% endhighlight %}

Pour l'entité `Affiliate`, nous souhaitons connaître uniquement la date de
création de la données.

{% highlight php %}
// src/Entity/Affiliate.php
// ...

use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

class Affiliate
{
    // ...

    public function setCreatedAtValue(LifecycleEventArgs $event)
    {
        $this->createdAt = new DateTime();
    }
}
{% endhighlight %}

> Signalons l'existence d'un bundle {% ext `StofDoctrineExtensionBundle`|https://goo.gl/EZWK72 %}
> contenant un ensemble d'extensions Doctrine pouvant être ajoutées à nos entités
> et possédant entre autre, une extension `Timestampable`. Cette dernière, permet
> de gérer de manière automatique les dates de création et de modification d'un
> entité sans avoir à ajouter manuellement les propriétés correspondantes.

Maintenant que nous avons indiqué à notre projet comment se connecter à notre
base de données, créé nos entités et indiqué la configuration nécessaire à la
liaison entre nos objets et le contenu de notre base, nous allons pouvoir 
initialiser cette dernière. Doctrine va encore nous faciliter le travail dans
cette tâche car l'ORM est distribué avec des commandes qui vont nous assister 
dans ce travail.

Dans un terminal, nous allons exéctuer les commandes suivantes :

{% highlight bash %}
$ bin/console doctrine:database:create # pour créer la base de données
Created database var/jobeet.db for connection named default


$ bin/console doctrine:schema:create # pour créer la structure des tables
ATTENTION: This operation should not be executed in a production environment.

Creating database schema...
Database schema created successfully!
{% endhighlight %}

> Vous pouvez constater que la base de données a été correctement initialiser
> en ouvrant le fichier contenant les données SQLite qui a été créé (`var/jobeet.db`)
> avec un outil tel que {% ext DB Browser for SQLite|http://sqlitebrowser.org %}.

Voilà qui conclut notre section d'introduction au modèle de données. Nous avons
maintenant une base de données (presque) prête à être utilisée et qui n'attends
plus que nos données.

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/03a-modele-donnees %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
