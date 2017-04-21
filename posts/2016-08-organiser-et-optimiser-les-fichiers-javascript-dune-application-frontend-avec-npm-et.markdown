--- 
title: Organiser et optimiser les fichiers javascript d'une application Frontend avec npm et Browserify 
published: true
pub_date: 2016-08-31 15:04:57 +0200
tags:
- AngularJS
- frontend
- browserify
- CommonJS
- npm
- javascript

---












![photo hors contexte](http://loutre.ch/sites/default/files/styles/9col__wide/public/img_3864.jpg?itok=mqi1KlkL)



> Ayant eu à développer une application AngularJS dernièrement, je te propose une manière d'organiser et de traiter le code de manière un peu plus efficace que de tout fourrer dans un seul et même fichier Javascript. Le début d'un projet de développement web implique effectivement quelques réflexions préliminaires quant à l'arborescence du code, s'il y a lieu de mettre en place un task runner pour automatiser certaines procédures, quel outil de gestion des dépendances utiliser, etc.




Cet article devra pouvoir te permettre, je l'espère, de mettre en place une application frontend. C'est pas vraiment un tutoriel à suivre, mais plutôt un article de compréhension générale parce que j'ai l'impression que c'est ce genre d'information qui manque sur le web. Ici, c'est AngularJS qui sera utilisé mais en pratique tu peux utiliser d'autres frameworks frontend.

## Introduction

Contrairement à certains systèmes backend écrits en Java ou en PHP, la plupart des frameworks frontend ne te forcent pas à utiliser un cycle de développement prédéfini. Avec Laravel (PHP) par exemple, l'arborescence est déjà établie (les contrôleurs ou les modèles par exemple ont leur dossier attribué) et on a des outils comme artisan et gulp qui permettent de gérer des tâches respectivement en backend et en frontend. Tu ne te poses pas de questions, il suffit de coder (et de bricoler quand la structure ne convient pas tout à fait).

AngularJS, pour sa part, est écrit en Javascript. La philosophie de ce langage est un peu différente de celle d'un language backend typique : en gros, on fait ce qu'on veut. Ça implique évidemment des risques du grand n'importe quoi, mais au final tu peux être trèèèès créatif, ce qui est plutôt cool pour faire des applications un peu innovantes. AngularJS est un framework qui propose des méchanismes très intéressants, mais pas de structure et d'organisation de code. C'est pour ça que tu trouveras, sur le web, une tonne d'exemples et de tutos avec des applications qui ne font qu'un fichier (app.js).

Ce qu'on va étudier dans cet article, c'est la sortie de ce concept de mono-fichier pour pouvoir construire quelque chose de modulaire et de flexible. On utilisera la notation CommonJS avec Browserify, on lancera des scripts npm pour compiler et minifier des fichiers.

## Dépendances externes
sdf
Dans mon cas, l'installation de toutes les librairies s'est faite avec npm. C'est un gestionnaire de dépendances très populaire pour le développement backend avec nodeJS mais quasiment toutes les dépendances frontend sont également disponibles.

## Système modulaire

Le réflexe qu'on trouve dans la plupart des tutos lorsqu'il s'agit de structurer les fichiers sources d'une application est de séparer les fonctions techniques (vues, contrôleurs, services, directives, etc.) dans des dossiers distincts. Ceci fonctionne bien lorsque l'application est simple et qu'elle ne propose que quelques fonctionnalités. Dès qu'on dépasse cette étape, il devient judicieux de coder avec des modules "métiers".

Ci-dessous, la structure du code par fonction technique.


```
	app/
	     controllers/
	         mainController.js
	         otherController.js
	     directives/
	         mainDirective.js
	         otherDirective.js
	     services/
	         userService.js
	         itemService.js
	     app.js
	views/
	     mainView.html
	     otherView.html
	     index.html
	node_modules/
```

C'est effectivement pratique quand t'as une petite application avec tout au plus trois ou quatre contrôleurs. Dès qu'on commence à dépasser ce stade, tu t'en sors plus dès qu'il faut chercher, ajouter ou désactiver une fonctionnalité. On va donc réorganiser notre code comme ça:

```
angular-app/
     modules/
         core/
             directive.js
             routes.js
             services.js
             ...
         admin/
         apicall/
         .../
     app.js
public/
     assets/
     views/
     index.html
theme/
     sass/
     theme.js
node_modules/
```

Comme tu peux le voir, on a deux aspects dans une application frontend: le fonctionnel et le visuel. Le code source de ces éléments est situé respectivement dans les dossiers `angular-app` et `theme`.

## Compilation du code

**Ou comment éviter d'avoir à charger moult fichiers dans le template html**

L'idée ici est de réutiliser les paradigmes de la programmation compilée. Bien que Javascript reste techniquement un langage interprété directement par le navigateur, on va _concaténer_ puis _minifier_ le code, on peut ainsi (par abus de langage certes) appeler cela de la compilation.

### Concaténation CommonJS

La concaténation consiste à ajouter bout à bout le contenu de plusieurs sources, que ce soit des chaînes ou des fichiers textes. Ce sont ces derniers qui vont être concaténés dans notre cas pour avoir au final qu'un seul fichier `bundle.js` dans le dossier `public/`.

Pour ce faire, on aurait la solution bête et facile d'utiliser... `cat`, l'utilitaire bash, avec `find` pour trouver les fichiers à traiter. Ça pourrait fonctionner mais on perd l'aspect _modulaire_ de l'histoire. On va donc utiliser la notation CommonJS avec Browserify, afin de pouvoir coder comme dans NodeJS.

On crée les modules CommonJS ainsi :

```javascript
var module = { };
// export du module
module.exports = module;
```

et on les réutilise comme ça:

```javascript
var module = require('./module');
```

Ca permet au final d'avoir un fichier `app.js` qui appelle d'autres fichiers, qui eux-mêmes en appellent d'autres, etc. La modularité est là : tu veux désactiver un module, il suffit de commenter la ligne correspondante au chargement du module (le `require()`).

Browserify est un utilitaire permettant de comprendre le concept CommonJS et de créer un seul fichier avec tous les appels intégrés, même ceux aux dépendances externes tels qu'AngularJS.

Lorsqu'on utilise `require('./nomduDossier')`, c'est le fichier `index.js` qui est recherché dans ce dossier. Tes modules peuvent donc tous avoir la même structure avec un fichier `index.js` qui est le point d'entrée du module, pouvant faire appel à des directives ou à des contrôleurs par exemple.

Un des avantages de Browserify est qu'il ne charge les fichier requis par require() qu'une seule fois même s'il apparaît à plusieurs endroits différents dans le code.

Voilà un exemple d'un des fichiers `index.js` :

```javascript
var dependencies = []; // liste des dépendances
require('angular')
    // Création d'un module 'monmodule' avec ses dépendances
    .module('monmodule', dependencies)
    // Ajout d'une directive
    .directive('moduleMyDirective', [require('./my.directive')]);
```

Il faudra donc créer un dossier s'appelant `my.directive` avec un fichier `index.js`, ou alors directement un fichier `my.directive.js` (l'extension n'est pas obligatoire avec `require()`).

### Minification

Une fois passé dans la moulinette browserify, le code peut être relativement lourd. On peut enlever les espaces blancs et les retours à la ligne pour gagner un peu de place, mais il est également possible de le rendre illisible par l'humain. `maBelleFonctionQuiAUnNomARallonge(parametre1, parametre2, parametre3)` peut simplement être traduit par `a(b,c,d)`, et c'est là qu'on gagne le plus d'octets. C'est le concept de l'_uglify_. Le fichier minifié et uglifyié (oui...) pourra ainsi s'appeler `bundle.min.js`.

Pour débugger l'application, vu que le code ne veut plus rien dire, tu peux passer des paramètres de _mapping_ à l'utilitaire de minification, afin d'avoir un fichier bundle.map. Ce fichier map est utilisé par les navigateur pour accéder au code original, mais n'est chargé que si c'est nécessaire, donc pas pour les visiteurs du site.

## Rendre ces tâches automatiques avec `npm`

Pour éviter d'effectuer ces différentes tâches à la main, deux logiciels principaux permettent leur automatisation : Grunt et Gulp. Ce sont les _Task Runners_ les plus connus actuellement. Tu peux aussi utiliser npm, et ainsi, étant donné que ce dernier est déjà installé et qu'il fait très bien le travail, on va utiliser l'utiliser pour éviter l'installation d'autres dépendances.

Ça fonctionne sous forme de scripts. On va donc créer ces derniers dans le fichier `package.json`, dans la section `scripts`:

```javascript
"scripts": {
        "build-js": "browserify -t browserify-ngannotate angular-app/app.js   debug | exorcist public/assets/js/bundle.js.map > public/assets/js/bundle.js",
        "build-min-js": "browserify -t browserify-ngannotate angular-app/app.js -p [minifyify   no-map] > public/assets/js/bundle.min.js",
 },
```

`exorcist` est utilitaire permettant de sortir le fichier map.js du fichier bundle.js, parce que le debug par défaut l'inclut et ça fait un fichier final très lourd. Je me dépêche d'écrire cet article parce que quelques mécanismes peuvent être améliorés avec Angular2 et on pourra en profiter pour en écrire un autre.

Au final, on a un fichier de base dans le dossier `angular-app`, qui appelle d'autres modules dans ce dossier et des dépendances externes. Ce fichier, grâce à Browserify, est concaténé dans un fichier bundle.js, qui lui-même est passé dans la machine de minification. Au final, on _exorcise_ le fichier et en sortent un fichier `bundle.min.js` prêt à l'emploi et un fichier `bundle.map`.

## Le futur est brillant

Angular2 apporte beaucoup de nouveautés en terme de performance et de fonctionnalités et est notamment compatible avec la nouvelle version de Javascript, ou plus justement d'ECMAScript, mais demande un petit temps d'adaptation pour le développeur. Angular2 permet également l'utilisation de NativeScript, un système utilisé pour créer des [applications mobiles Android et IOS natives](https://www.smashingmagazine.com/2016/07/cross-platform-native-apps-single-code-set-telerik-nativescript/).

Une des fonctionnalités très attendue par la communauté de développeurs dans la nouvelle version de la norme – nommée ES6 ou encore ES2015 – est le système de gestion de modules. Une syntaxe finale pour l'import et l'export de fonctionnalités entre les fichiers a été mise en place et il pourrait être intéressant d'utiliser ce concept pour remplacer le système modulaire du couple CommonJS/Browserify. Ceci permettrait d'optimiser grandement la minification du code grâce au concept de [Tree-Shaking](https://www.linkedin.com/pulse/forget-browserify-webpack-embrace-rollupjs-michel-ruffieux?trk=prof-post) (on garde que les fonctionnalités utilisées) de la libraire [rollup.js](http://rollupjs.org). Il faudrait néanmoins revoir la manière d'installer des dépendances externes.





















# Punch blog



Thanks for trying out Punch blog. You start writing editing this post itself. Just go to `posts/` directory and you will find a file named `1970-01-01-lets-start.markdown`. Rename it with today's date and your post title. Then open it, delete this post and start writing your own.

Before we go any further, here's cute picture of a cat to cheer you up :)

![cat](/img/cat.jpg)

At the top of this file, you will find a YAML front matter block (similar to [Jekyll](https://github.com/mojombo/jekyll)). You can define any variable you want to use in templates. Punch will look for the following variables when generating the site - `title`, `published` and `tags`.

You can set any blog specific settings under the section `blog` in the `config.json`. Default blog settings can be found [here](https://github.com/laktek/punch-blog-content-handler). Apart from the default settings, there are two custom settngs for this project, which define the number of posts (`homepage_posts`) and paragraphs (`teaser_length`) to show in homepage. Here are the default `blog` section from `config.json`.

```javascript
	"blog": {
		"posts_dir": "posts",
		"post_format": "markdown",	
		"post_url": "/{year}/{month}/{date}/{title}",
		"teaser_length": 2,
		"homepage_posts": 10,

		"archive_urls": {
			"all": "/archive",
			"year": "/{year}",
			"year_month": "/{year}/{month}",
			"year_month_date": "/{year}/{month}/{date}",
			"tag": "/tag/{tag}"
		}
	}
```

Punch-blog uses [Prism.js](http://prismjs.com/) for automatic syntax highlighting. There are several other nifty features like that on Punch-blog. Here's a full list: 

* Preview posts, as you write them.
* Easily publish to Amazon S3.
* Pretty URLs for permalinks (no .html, configurable).
* Responsive, customizable theme based on [HTML5Boilerplate](html5boilerplate.com) & [320andup framework](https://github.com/malarkey/320andup/).
* Load fonts from multiple sources with [WebFonts Loader](https://github.com/typekit/webfontloader).
* Easily configure Google Analytics, Tweet button & Disqus comments.
* Highlighting the current page link.
* Post archives by tags.
* Post archives by year, month or date.
* Write posts using GitHub flavored Markdown.
* Client-side code highlighting with Prism.js
* Published/draft states.
* Automatically minifies and bundles JavaScript/CSS.
* RSS feed 
* Sitemap.xml

Also, you can use any other features available in Punch.

* Manage other pages with Punch's default content handler.
* Extend the behavior by writing your own helpers.

This is so sexy and awesome right? Yes, just like the Gangnam style:

<iframe width="560" height="315" src="http://www.youtube.com/embed/9bZkp7q19f0" frameborder="0" allowfullscreen></iframe> 


Happy Blogging!
