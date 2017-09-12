---
layout: post
title: "Tutorial Jobeet pour Symfony 4 - Partie 2: Le projet"
menu: blog
priority: 0.80
type: jobeet
---

Nous avons précédemment créé la structure de notre projet Symfony et préparé
notre environnement de développement. Nous allons très prochainement pouvoir
commencer à écrire nos premières lignes de code. Mais avant cela, attardons-
nous un peu sur les exigences de notre projet.

> Ce billet est principalement d'une recopie des exigences fonctionnelles du
> {%ext tutorial initial|https://symfony.com/legacy/doc/jobeet/1_4/fr/02?orm=Doctrine %}.
> Je vous recommande vivement de lire ce dernier car des explications sur les
> nouvelles pratiques liées à Symfony 4 y sont décrites.

Jobeet est un logiciel open-source de recherche d'emploi qui ne fait qu'une
seule chose, mais le fait bien. Il est facile d'utilisation, à adapter, à faire
évoluer, et à intégrer à votre site internet. Il est multi langues dès le départ,
et bien sûr il utilise les dernières technologies du web 2.0 pour améliorer
l'expérience utilisateur. Il fournit également des feeds et une API pour
interagir avec lui en programmant.

Le site web Jobeet a quatre types d'utilisateurs :

* L'administrateur : il est propriétaire du site et il a le pouvoir magique
* L'utilisateur : il visite le site pour chercher un emploi
* L'annonceur : il soumet une offre d'emploi
* L'affilié : il re-édite certains emplois sur son site

Nous allons maintenant découper le projet en scénarios d'utilisation. L'application
est séparée en deux interfaces : le frontend (où les utilisateurs interagissent
avec le site), et le backend (où les administrateurs gèrent le site).

Symfony recommande maintenant et générera dorénavant des applications orientées
bundle-less. Si vous débutez avec Symfony, un {% ext bundle|https://symfony.com/doc/current/bundles.html %}
peut être considéré comme un "plugin". Ils permettent de regrouper du code qui
est regroupé par fonctionnalité et redistribuable dans n'importe quel projet Symfony.

Nous allons maintenant lister les exigences fonctionnelles de notre application.
Ces dernières seront numérotés et débuteront par un `F` pour les aspects frontend
et par un `B` pour les aspects backend.

### F1: En tant qu'utilisateur, je vois dernières offres actives sur la page d'accueil

Quand un utilisateur vient sur le site Jobeet, il voit une liste des emplois actifs.
Les emplois sont classés par catégorie, puis par date de publication (emplois plus
récents en premier). Pour chaque emploi, seul le lieu, la position, et la société
sont affichés.

Pour chaque catégorie, la liste ne montre que les 10 premiers emplois et un lien
permet de lister tous les emplois pour une catégorie donnée (scénario F2).

Sur la page d'accueil , l'utilisateur peut affiner la liste des travaux (scénario
F3), ou par la soumission d'un nouvel emploi (scénario F5).

### F2: En tant qu'utilisateur, je peux lister les emplois d'une catégorie

Quand un utilisateur clique sur un nom de catégorie ou sur le lien "more jobs"
sur la page d'accueil, il voit tous les emplois pour cette catégorie triée par
date.

La liste est paginée avec 20 emplois par page.

### F3: En tant qu'utilisateur, je peux affiner les offres d'une catégorie

L'utilisateur peut saisir quelques mots-clés pour affiner sa recherche. Les mots-
clés peuvent être des mots trouvés dans l'emplacement, la position, la catégorie,
ou les champs de l'entreprise.

### F4: En tant qu'utilisateur, je peux visualiser le détail d'une offre

L'utilisateur peut sélectionner un emploi dans la liste pour afficher des
informations plus détaillées.

### F5: En tant qu'utilisateur, je peux soumettre une offre d'emploi

Un utilisateur peut soumettre un emploi (il n'est pas nécessaire de devoir
créer un compte). Un emploi est composé de plusieurs informations.

Le processus est simple, avec seulement deux étapes : d'abord, l'utilisateur
remplit dans le formulaire toutes les informations nécessaires pour décrire
l'emploi, puis il valide les informations en visualisant la page finale de
l'emploi.

Même si l'utilisateur n'a pas de compte, un emploi peut être modifié ultérieurement,
grâce à une URL spécifique (protégé par un jeton donné à l'utilisateur lorsque
l'emploi est créé).

Chaque poste de travail est en ligne pendant 30 jours (ce qui est configurable
par l'administrateur - voir Histoire B2). Un utilisateur peut revenir réactiver
ou prolonger la validité de l'annonce pour un supplément de 30 jours, mais
seulement lorsque le travail expire dans moins de 5 jours.

### F6: En tant qu'utilisateur, je peux affilier ma société

Un utilisateur doit demander à devenir un affilié et être autorisés à utiliser
l'API Jobeet. Pour postuler, il doit donner des informations.

Le compte d' un affilié doit être activé par l'administrateur (scénario B3).
Une fois activé, l'affilié reçoit un jeton pour une utilisation avec l'API par
email.

Lors de l'application, l'affilié peut également choisir d'obtenir des emplois
auprès d'un sous-ensemble des catégories disponibles.

### F7: En tant qu'utilisateur, je peux exporter  la liste des emplois actifs

Un affilié peut récupérer la liste des emplois en cours en appelant l'API avec
le jeton de sa filiale. La liste peut être retournée en XML, JSON ou YAML.

La liste contient les informations publiques disponibles pour un emploi.

L'affilié peut également limiter le nombre d'emplois qui seront retournés,
et d'affiner sa requête en spécifiant une catégorie.

### B1: En tant qu'administrateur, je peux configurer le site

L'administrateur peut modifier les catégories disponibles sur le site.

### B2: En tant qu'administrateur, je peux administrer les offres d'emploi

Un administrateur peut modifier et supprimer tous les emplois affichés.

### B3: En tant qu'administrateur, je peux gérer les affiliés

L'administrateur peut créer ou modifier des affiliés. Il est responsable de
l'activation d'un affilié et peut également les désactiver.

Lorsque l'administrateur active une nouvelle filiale, le système crée un
jeton unique à utiliser par la filiale.

> Retrouvez tous les tutorials Jobeet disponibles depuis le [billet d'introduction
> de la série]({% post_url 2017-09-12-tutorial-jobeet-symfony-4-introduction %}).
> Le code source de cette application est également disponible sur
> {% ext Github|https://github.com/jdecool/jobeet/tree/02-projet %}.
> Vous trouvez une branche associée à l'état du projet après chaque chapitre.
