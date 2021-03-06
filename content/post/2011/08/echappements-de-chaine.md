+++
date = "2011-08-18T19:01:00.000000+02:00"
title = "Échappement de chaînes"
tags = ["PostgreSQLFr"]
categories = ["PostgreSQL","PostgreSQLFr"]
thumbnailImage = "/img/old/postgresqlfr-logo.png"
thumbnailImagePosition = "left"
coverImage = "/img/old/postgresqlfr-logo.png"
coverSize = "partial"
coverMeta = "out"
aliases = ["/blog/2011/08/18-echappements-de-chaine",
           "/blog/2011/08/18-echappements-de-chaine.html"]
+++

Parmis les nouveautés de la 
[prochaine version](http://www.postgresql.org/about/news.1331) de 
[PostgreSQL](http://www.postgresql.org/), la fameuse 
`9.1`,
il faut signaler le changement de valeur par défaut de la variable
`standard_conforming_strings`, qui passe à 
*vraie*.

En effet, l'utilisation d'échappements avec le caractère « anti-slash »
n'est pas conforme au standard SQL.  Le paramètre
`standard_conforming_strings` permet de contrôler le comportement de
PostgreSQL lorsqu'il lit une chaîne de caractère dans une requête SQL.

Voyons quelques exemples :

~~~
dimitri=# set standard_conforming_strings to true;
SET
dimitri=# select 'hop''';
 ?column? 
----------
 hop'
(1 ligne)

dimitri=# select 'hop\'';
dimitri'# ';
 ?column? 
----------
 hop\';
 
(1 ligne)

dimitri=# select E'hop\'';
 ?column? 
----------
 hop'
(1 ligne)

dimitri=# set standard_conforming_strings to false;
SET
dimitri=# select E'hop\'';
 ?column? 
----------
 hop'
(1 ligne)

dimitri=# select 'hop\'';
ATTENTION:  utilisation non standard de \' dans une cha&#xEE;ne litt&#xE9;rale
LIGNE 1 : select 'hop\'';
                 ^
ASTUCE : Utilisez '' pour &#xE9;crire des guillemets dans une cha&#xEE;ne ou utilisez
la syntaxe de cha&#xEE;ne d'&#xE9;chappement (E'...').
 ?column? 
----------
 hop'
(1 ligne)
~~~


Il existe un moyen de forcer PostgreSQL à accepter l'utilisation
d'échappements avec « anti-slash » indépendamment de la valeur de
`standard_conforming_strings`, c'est la notation préfixée avec 
`E`.  Il est
recommandé de toujours l'utiliser dès lors que la chaîne de caractère
contient des « anti-slash » utilisés comme échappement (du caractère simple
guillemet en général).

Le paramètre 
`escape_string_warning`, enfin, permet de désactiver les
avertissements tels que présentés dans le dernier exemple ci-dessus,
lorsqu'il est positionné à 
`off`.  Bien sûr, sa valeur par défaut est 
`on`.

Toute apparition de ce 
*WARNING* lorsque 
`escape_string_warning` est 
`on` signifie
que votre application n'est pas prête à migrer à 
`9.1` avec son paramétrage
par défaut.  Il existe deux actions possible : changer le paramétrage de sa
nouvelle valeur par défaut à sa précédente, ou bien corriger ses
applications pour utiliser le préfixe 
`E` dès que cela est nécessaire.

L'utilisation de 
`standard_conforming_strings` à 
`on` présente un autre avantage
au respect du standard SQL : la sécurité contre les injections.  S'il n'est
pas possible d'échapper le guillemet simple qui termine toute chaîne de
caractère utilisateur, il devient compliqué de jouer au plus malin avec le
*parser*.  Le mieux ici reste bien sûr d'utiliser les requêtes paramétrées, à
suivre dans un prochain article.
