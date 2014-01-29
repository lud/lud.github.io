---
title: "Des libraries Erlang globales"
layout: "posts"
lang: fr
categories: erlang
---


# Gérer ses librairies Erlang de façon globale


Depuis quelques années, les développeurs Erlang disposent de
[Rebar](https://github.com/basho/rebar) pour gérer les dépendances de leur
projet facilement. Cet outil permet également de créer des exécutables, de
générer des *releases*, et, plus généralement, de gérer facilement le
développement d'une application.

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
Ici nous allons avoir en local les applications suivantes : `gproc`, `lager`,
`pmod_transform` et `parse_trans`. Voici le contenu de ce fichier :

{% highlight erlang %}
{deps, [ 

  {gproc,          ".*", 
  {git, "https://github.com/uwiger/gproc.git", "HEAD"}}

, {lager,          ".*", 
  {git, "https://github.com/basho/lager.git", "HEAD"}}

, {pmod_transform, ".*", 
  {git, "https://github.com/erlang/pmod_transform.git", "HEAD"}}

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
dépendances peuvent également avoir leurs propres dépendances. Dans notre cas,
`gproc` a besoin de `edown` et `lager` de `goldrush`. Rebar est donc d'une
grande aide ici.

Un autre avantage de l'utilisation de Rebar est de pouvoir indiquer le *tag*
ou le *commit* précis de la librairie. Cependant, si l'on veut avoir la même
dépendance en plusieurs versions, il faudra toujours passer par un peu de
gestion manuelle. Dans notre cas, nous nous contentons du `HEAD` de chaque
librairie.


### Inclure les librairies au *path*


Cette seconde étape est encore plus simple, elle consite à charger
automatiquement ces libraires lors du lancement du *runtime* Erlang.

Lors du démarrage, le *runtime* cherche automatiquement un fichier `.erlang`
dans le dossier courant, puis dans le *home* de l'utilisateur s'il n'en trouve
pas.

Nous allons donc créer ce fichier dans notre *home* :

{% highlight bash %}
cd
touch .erlang
{% endhighlight %}

Ce fichier attend des expressions comparables à celles que l'on entre dans le
shell Erlang. Voici le contenu du fichier pour charger ces librairies :

{% highlight erlang %}io:format("Chargement des librairies ... ").
LoadLib = 
  fun(Dir) -> 
    case code:add_path("/home/lud/src/erllib/deps/" ++ Dir ++ "/ebin")
     of true -> 
        ok
     ; {error,Error} -> 
        io:format("Erreur du chargement de ~p: ~p~n",[Dir,Error])
    end
  end.
[LoadLib(X) || X <- [
  "edown",
  "goldrush",
  "gproc",
  "lager",
  "parse_trans",
  "pmod_transform"
]]. 
io:format("ok~n").
{% endhighlight %}

Plusieurs chose concernant ce code :

* Ne pas oublier les dépendances de nos dépendances ! (`edown` et `goldrush`)
* Ici, j'ai vonlontairement utilisé une `fun` et une compréhension de liste pour montrer qu'on est assez libre de faire ce qu'on veut dans ce fichier. Tout comme dans le *shell*, il n'est pas possible d'y définir des fonctions, d'où l'utilisation de cette `fun`. 
* La fun est un peut grande, dans notre cas on est sûrs de nos chemins, donc un simple `code:add_path` est nécessaire. Mais dans le cas ou nos librairies sont éparpillées dans plusieurs dossiers, c'est toujours utile de savoir qu'on s'est planté dans un chemin !
* Notez que pour chaque dépendance, c'est le dossier `ebin` qui est visé, et non la racine du projet. Certains projets peuvent également avoir leurs fichiers `.beam` dans un dossier nommé diféremment, ou bien dans plusieurs dossiers.

### L'heure du test !

Au lancement du shell, on devrait donc avoir le résultat suivant :

{% highlight bash %}
$ cd /dans/nimporte/quel/dossier
$ erl
Erlang R16B01 (erts-5.10.2) [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Chargement des librairies ... ok
Eshell V5.10.2  (abort with ^G)
1> lager:start().
ok
2> 00:38:05.130 [info] Application lager started on node nonode@nohost
application:start(gproc).
00:38:14.369 [info] Application gproc started on node nonode@nohost
ok
... etc ...
{% endhighlight %}

Nous testons le démarrage des applications `lager` et `gproc` et tout semble
fonctionner comme il se doit.

Comme je le disais, le *runtime* va d'abord chercher un fichier `.erlang` dans
le dossier courant. Si l'on souhaite, pour un certain projet, utiliser
d'autres versions des dépendances, il suffit alors de les télécharger comme
nous l'avons fait dans un autre dossier que `erllib`, et d'écrire un fichier
`.erlang` spécifique dans notre projet.

Attention toutefois, si vous partagez vos applications basées sur Rebar en
ligne, par exemple sur Github, la manière classique (dépendances gérées
directement dans le projet) reste à privilégier si vous voulez que d'autres
développeurs puissent contribuer.

Voilà, c'était l'astuce facile du jour !
