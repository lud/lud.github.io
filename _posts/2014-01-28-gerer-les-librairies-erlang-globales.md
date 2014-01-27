---
title: "Des libraries Erlang globales"
layout: "posts"
lang: fr
categories: erlang
---


# Gérer ses librairies Erlang de façon globale


Depuis quelques années, les développeurs Erlang disposent de Rebar pour gérer
les dépendances de leur projet facilement. Cet outil permet également de créer
des exécutables, de générer des *releases*, et, plus généralement, de gérer
facilement le développement d'une application.

Il existe d'autres outils équivalents à Rebar, la petite manipulation toute
simple que je montre ici s'adapte facilement à n'importe quel autre outil.
Cela peut même être fait manuellement ou avec git.


## Factoriser les dépendances


Une application erlang s'appuie généralement sur d'autres applications. Elles
sont parfois disponibles dans le système de base, comme Mnesia ou Crypto (bien
que celles-ci puissent faire partie de paquets spécifiques), parfois il faut
aller faire son marché sur Github.

Pour gérer ses dépendances avec Rebar, il faut spécifier les dépendances dans
le fichier `rebar.config`. Si on souhaite utiliser `gproc` dans trois
applications différentes, chacune d'entre elles aura une copie de `gproc` dans
son dossier `deps`.

Cela peut donc vite devenir encombrant. Quand on développe soi-même une
application A qui sert à plusieurs de nos autres projets (B,C,D) et qu'on veut
faire une modification dessus, il faut :

1. Faire la modification dans notre application A
2. Aller dans notre autre projet B qui l'utilise en tant que dépendance
3. Relance un petit `rebar get-deps` afin de la copier dans `deps`

Et si notre projet B tracke notre application A depuis Github : 

1. Faire la modification dans notre application A
2. Commiter et envoyer sur Github
3. Aller dans notre autre projet B qui l'utilise en tant que dépendance
4. Relance un petit `rebar get-deps` afin de la copier dans `deps`

C'est ... relou, voilà. Une raison de plus pour n'avoir qu'une seule copie de
A en local. On peut la commiter *a posteriori*.


## Une application pour les gouverner toutes


Nous allons donc créer une nouvelle application, qui ne servira qu'à gérer nos
applications tierces.


### Télécharger toutes les dépendances en local


Il faut commencer par choisir un dossier approrié. Personnellement j'ai un
dossier `src` dans mon dossier *home* qui contient tous mes projets. Ensuite,
nous créons on dossier contenant l'application rebar, je le nomme `erllib` :

{% highlight bash %}
    cd
    cd src
    mkdir erllib
    cd erllib
{% endhighlight %}

Ensuite, créer le fichier de configuration pour Rebar.

{% highlight bash %}
    touch rebar.config
{% endhighlight %}

Il faut ensuite éditer ce fichier de configuration avec votre éditeur favori.
Ici nous allons avoir en local les applications suivantes : `gproc`,
`pmod_transform` et `parse_trans`. Voici le contenu de ce fichier :

{% highlight erlang %}
    {deps, [ 

        {pmod_transform, ".*", 
        {git, "https://github.com/erlang/pmod_transform.git", "HEAD"}}

      , {gproc,          ".*", 
        {git, "https://github.com/uwiger/gproc.git", "HEAD"}}

      , {parse_trans,    ".*", 
        {git,  "https://github.com/uwiger/parse_trans.git", "HEAD"}}
    ]}.
{% endhighlight %}


Enfin, lancer le téléchargement des librairies et les compiler avec rebar.

{% highlight bash %}
    rebar get-deps compile
{% endhighlight %}

Voilà, les librairies sont disponibles en local. Sans Rebar, on peut cloner
chaque *repository* à la main et les placer dans un dossier commun. Mais les
dépendances peuvent également avoir leurs propres dépendances (`grpoc` a
besoin de `edown` par exemple). Rebar est donc d'une grande aide ici.


### Inclure les librairies au *path*


... à venir