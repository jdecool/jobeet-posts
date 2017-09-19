---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 3B: Les données initiales"
menu: blog
priority: 0.80
type: jobeet
---

Nous avons donc créé notre base et données avec les tables qui permettrons de
stocker les informations que notre application va manipuler. Avant de commencer
l'implémentation des fonctionnalités évoquées dans les billets précédents, nous
allons commencer par insérer un jeu de données initiales (également appelées
**fixtures**) afin de ne pas démarrer avec un projet vide.

Pour créer nos données, nous allons utiliser un bundle également fourni par les
équipes de Doctrine et qui propose un moyen de charger des données en base. Nous
avions jusqu'à maintenant utilisé uniquement des modules officiels de Symfony.
Le bundle que nous allons utiliser (`DoctrineFixturesBundle`) est un module non
officiel de Symfony. Or, par défaut, Symfony Flex ne permet de ne travailler
qu'avec les bundles officiels de Symfony.

Il est néanmoins possible d'utiliser Flex avec des bundles, mais il sera nécessaire
de le spécifier explicitement dans la configuration de ce dernier. Pour cela, nous
allons modifier la valeur du paramètre `allow-contrib` présent dans notre fichier
`composer.json` au travers de la commande suivante :

{% highlight bash %}
$ composer config extra.symfony.allow-contrib true
{% endhighlight %}

Il est maintenant possible de télécharger la dépendance. Cette dernière n'étant
utile que pour du développement, nous allons l'installer comme une dépendance
de développement :

{% highlight bash %}
$ composer require doctrine/doctrine-fixtures-bundle --dev

Using version ^2.4 for doctrine/doctrine-fixtures-bundle
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 2 installs, 0 updates, 0 removals
  - Installing doctrine/data-fixtures (v1.2.2): Loading from cache
  - Installing doctrine/doctrine-fixtures-bundle (v2.4.0): Loading from cache
Writing lock file
Generating autoload files
Symfony operations: 1 recipe
  - Configuring doctrine/doctrine-fixtures-bundle (2.4): From github.com/symfony/recipes-contrib:master
Executing script make cache-warmup [OK]
Executing script assets:install --symlink --relative public [OK]
{% endhighlight %}

L'installation de cette dépendance va créer un dossier `src/DataFixtures/ORM` qui
contiendra nos déclarations de fixtures. Mais avant d'écrire ces dernières, vous
avez peut-être remarqué, que lorsque nous avons écrit nos entités, nous avons
déclaré les propriétés de nos objets comment étant `private`. Nous allons donc
devoir commencer par écrire les méthodes `get` qui permettront d'accéder à nos
propriétés ainsi que les méthodes `set` qui permettront de modifier la valeur
de nos propriétés.

> L'activation du paramètre `allow-contrib` notamment permis la création du
> répertoire qui accueillera nos fixtures. Même si nous n'avions pas changé
> cette configuration, il aurait été possible d'installer le bundle, mais nous
> n'aurions pas profité des mécanismes de Flex qui permettent de créer une
> configuration du bundle par défaut.

Si vous avez eu la curiosité de regarder la liste des commandes proposées par le
bundle Doctrine, vous aurrez peut-être remarqué qu'il existe une commande
`doctrine:generate:entities` dont le rôle est justement de remplir cette tâche.

Notre projet suit la convention {% ext PSR-4| http://www.php-fig.org/psr/psr-4 %}
décrivant la norme sur le chargement des fichiers PHP en fonction de leur arborescence
(appelée {% ext autoloading|http://php.net/autoload %}). Malheureusement, le
générateur de Doctrine n'est actuellement pas compatible avec cette norme (et ce
n'est pas en projet).

Mais cela n'est pas un problème étant donnée que la plupart des outils de développement
(tel que PHPStorm, Netbeans, Eclipse, Sublime Text, Atom, VSCode, ...) ont une
fonctionnalité (ou des plugins) permettant de générer du code automatiquement et
notamment nos getters et setters.

Nous allons donc générer des méthodes `get` pour l'ensemble des propriétés de
nos entités ainsi que les méthodes `set` associées, à l'exception des propriétés
`$id` (car une clé primaire ne doit pouvoir être modifié), `createdAt` et
`updateAt` (car nous avons déjà ajouté les méthodes permettant de modifier ces
données).

> Si vous souhaitez gagner du temps, vous pouvez accéder au code source correspondant
> à ce billet sur {% ext Github|https://github.com/jdecool/jobeet/tree/04-donnes-initiales %}
> pour copier/coller les méthodes décrites ci-dessus.

Nous pouvons maintenant commencer à écrire nos fixtures, c'est-à-dire les classes
qui vont s'occuper de la création notre jeu de données initial. Nous allons
commencer par créer des catégories d'offre d'emploi en créant une classe
`LoadCategoryData` dans le dossier `src/DataFixtures/ORM` :

{% highlight php %}
<?php //src/DataFixtures/ORM/LoadCategoryData.php

namespace App\DataFixtures\ORM;

use App\Entity\Category;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;

class LoadCategoryData extends AbstractFixture implements OrderedFixtureInterface
{
    public function load(ObjectManager $manager)
    {
        $design = new Category();
        $design->setName('Design');
        $manager->persist($design);

        $programming = new Category();
        $programming->setName('Programming');
        $manager->persist($programming);

        $managing = new Category();
        $managing->setName('Manager');
        $manager->persist($managing);

        $administrator = new Category();
        $administrator->setName('Administrator');
        $manager->persist($administrator);

        $manager->flush();

        $this->addReference('category-design', $design);
        $this->addReference('category-programming', $programming);
        $this->addReference('category-managing', $managing);
        $this->addReference('category-administrator', $administrator);
    }

    public function getOrder()
    {
        return 1;
    }
}
{% endhighlight  %}

Pour charger des données, notre classe doit étendre de `AbstractFixture` qui
contient des fonctions de base de gestion de fixtures. Nous implémentons ensuite
une interface `OrderedFixtureInterface` permettant de spécifier un ordre d'
insertion des nos fixtures (dans le cas présent, nous ne pouvons enregistrer
une offre d'emploi si nous n'avons pas auparavant créé des catégories) au travers
de la méthode `getOrder()`.

Le chargement des données se fait au sein d'une méthode `load(ObjectManager $manager)`.
Il s'agit de la méthode appelée lorsque les données doivent être insérées. Cette
méthode prend en paramètre un objet de type `ObjectManager` qui est l'objet de
Doctrine qui permet de travailler avec la base de données.

Ce dernier permet d'appeler la méthode `persist` pour indiquer à Doctrine que
nous souhaitons persister un objet en base. Pour que la donnée soit réellement
écriture en base on utilise un appel à la méthode `flush`. Dans le cas où l'on
fait plusieurs appels à la méthode `persist` (comme c'est le cas ici) et qu'ensuite
on `flush`, les insertions en bases sont réalisés au sein d'une transaction.

Une fois les données enregistrées, nous souhaitons les rendre accessibles depuis
nos autres fixtures pour par exemple pouvoir affecter une catégorie à une offre
d'emploi. Pour cela, nous allons définir explicitement les objets auxquels nous
souhaitons accéder en appelant la méthode `addReference`.

Nous pouvons ensuite passer à notre fixture suivante :

{% highlight php %}
<?php  //src/DataFixtures/ORM/LoadJobData.php

namespace App\DataFixtures\ORM;

use App\Entity\Job;
use Doctrine\Common\DataFixtures\AbstractFixture;
use Doctrine\Common\DataFixtures\OrderedFixtureInterface;
use Doctrine\Common\Persistence\ObjectManager;

class LoadJobData extends AbstractFixture implements OrderedFixtureInterface
{
    public function load(ObjectManager $manager)
    {
        $categoryProgramming = $this->getReference('category-programming');
        $categoryDesign = $this->getReference('category-design');

        $jobSensioLabs = new Job();
        $jobSensioLabs->setCategory($categoryProgramming);
        $jobSensioLabs->setType('full-time');
        $jobSensioLabs->setCompany('Sensio Labs');
        $jobSensioLabs->setLogo('sensio-labs.gif');
        $jobSensioLabs->setUrl('http://www.sensiolabs.com/');
        $jobSensioLabs->setPosition('Web Developer');
        $jobSensioLabs->setLocation('Paris, France');
        $jobSensioLabs->setDescription("You've already developed websites with symfony and you want to work with Open-Source technologies. You have a minimum of 3 years experience in web development with PHP or Java and you wish to participate to development of Web 2.0 sites using the best frameworks available.");
        $jobSensioLabs->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
        $jobSensioLabs->setIsPublic(true);
        $jobSensioLabs->setIsActivated(true);
        $jobSensioLabs->setToken('job_sensio_labs');
        $jobSensioLabs->setEmail('job@example.com');
        $jobSensioLabs->setExpiresAt(new \DateTime('2012-10-10'));
        $manager->persist($jobSensioLabs);

        $jobExtremeSensio = new Job();
        $jobExtremeSensio->setCategory($categoryDesign);
        $jobExtremeSensio->setType('part-time');
        $jobExtremeSensio->setCompany('Extreme Sensio');
        $jobExtremeSensio->setLogo('extreme-sensio.gif');
        $jobExtremeSensio->setUrl('http://www.extreme-sensio.com/');
        $jobExtremeSensio->setPosition('Web Designer');
        $jobExtremeSensio->setLocation('Paris, France');
        $jobExtremeSensio->setDescription('Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in.');
        $jobExtremeSensio->setHowToApply('Send your resume to fabien.potencier [at] sensio.com');
        $jobExtremeSensio->setIsPublic(true);
        $jobExtremeSensio->setIsActivated(true);
        $jobExtremeSensio->setToken('job_extreme_sensio');
        $jobExtremeSensio->setEmail('job@example.com');
        $jobExtremeSensio->setExpiresAt(new \DateTime('2012-10-10'));
        $manager->persist($jobExtremeSensio);

        $manager->flush();
    }

    public function getOrder()
    {
        return 2;
    }
}
{% endhighlight %}

> Notez dans le code ci-dessus l'utilisation de la méthode `getReference` nous
> permettant de récupérer une donnée qui a été créée dans la fixture précédente.

Il ne reste maintenant plus qu'à charger nos données au travers de la commande :

{% highlight bash %}
$ bin/console doctrine:fixtures:load
Careful, database will be purged. Do you want to continue y/N ?y
  > purging database
  > loading [1] App\DataFixtures\ORM\LoadCategoryData
  > loading [2] App\DataFixtures\ORM\LoadJobData
{% endhighlight %}

Et voilà !

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/03b-donnees-initiales %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
