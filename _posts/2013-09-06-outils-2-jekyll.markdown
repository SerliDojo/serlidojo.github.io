---
layout: post
title:  "Outils #2 - Jekyll"
date:   2013-09-06 21:30:00
comments: true
---

Le deuxième épisode de la série Outils est arrivé et a mis à l'honneur Jekyll ([http://www.jekylrb.com](http://www.jekylrb.com)), un moteur de site Web statique utilisé notamment par GitHub.


Le défi
----------

Cette fois, le défi consistait à initialiser et mettre en ligne un blog pour le dojo. 
Ce blog devra permettre de publier les comptes-rendus et d'annoncer les différentes séances.


Un site statique orienté blog
----------

Mélanger un peu de langage wiki, un peu de templating et quelques fonctionnalités de blog comme les permaliens, les catégories et les billets ; secouer (sans frapper) et server le tout en ruby. Vous obtiendrez un site statique simple, rapide, fonctionnel.

En effet, Jekyll va associer un langage de contenu comme le Textile à un langage de template, le Liquid. Il s'appuie sur cette combinaison pour fournir un moteur de blog minimaliste basé sur des principes statiques comme les noms de fichiers.


Une installation basée sur Ruby
----------

Jekyll s'appuie sur Ruby ainsi que quelques modules supplémentaires pour construire les sites. 

Par défaut, il ne supporte que les plates-formes Unix. En effet, l'installation de Ruby et de ces modules n'est pas toujours très intuitives sous Windows.

Le Coding Dojo se voulant proche des conditions de travail réelles, nous avons tenté l'aventure sous ce dernier système.

[Quelques tutos](http://www.madhur.co.in/blog/2011/09/01/runningjekyllwindows.html) plus tard, nous avions un moteur Jekyll tout prêt, mais sans le module Pygments, nécessaire à la coloration syntaxique, la partie liée à Python n'ayant pas été couronnée de succès).


Une structure de projet légère
----------

Lorsque l'on démarre un site Jekyll, on commence par une petite commande `jekyll new dojo-serli`. On obtient alors une structure de répertoire assez simple, composée de seulement trois dossiers (_layouts, _posts et css).

![Structure du projet après la création](/img/jekyll/structure-creation.png)

On remarque d'emblée que les dossiers et fichiers débutant par un `_` seront supprimés du site final. D'ailleurs, après l'exécution de la commande suivante, `jekyll build`, nous obtenons un dossier supplémentaire (_site) contenant le résultat de la construction du site.

On y retrouve nos ressources statiques (CSS) ainsi que les pages définitives comme le fichier index.html. On y trouve également les fameux billets de blog, regroupés dans des sous-dossiers par catégorie, année, mois et jour.

![Structure du projet après la compilation](/img/jekyll/structure-compilation.png)

Le site peut maintenant être déployé sur un serveur quelconque comme Apache, toutes les pages seront consultables statiquement. Les billets seront alors accéder par une URL simple comme `/jekyll/update/2013/09/06/welcome-to-jekyll.html` pour le billet par défaut.


La configuration en YAML
----------

La première modification que nous avons apporté à ce projet par défaut est sa configuration. Cette dernière se trouve simplement dans un fichier _config.yml.

Ce fichier ne contient pour l'instant que le nom du site, l'activation de Pygments (que nous avons désactivé, voir plus haut) ainsi que le moteur Markdown.

```yaml
name: Your New Jekyll Site
markdown: redcarpet
pygments: false
```

Nous retrouvons également un peu de YAML en en-tête de chacun des fichiers interprétés par Jekyll. Cet en-tête permet à Jekyll d'identifier le layout à utiliser ainsi que quelques autres méta-données. Ils appellent cela le front-matter.

```yaml
---
layout: post
title:  "Outils #2 - Jekyll"
date:   2013-09-06 21:30:00
---
```


Du contenu simple et explicite
----------

L'idée de base est donc de produire du contenu dans un langage simple et courant, comme ceux utilisés dans les wiki. Le moteur de Jekyll pourra ensuite l'interpréter et le transformer en HTML statique.

Jekyll accepte le langage wiki Textile, ainsi que le langage Markdown. Mais il accepte également de traiter les fichiers écrits en lanage HTML. C'est ainsi que le fchier index.html est interprété. Nous avons donc commencé par éditier ce fichier. 

Malgré que le langage HTML soit utilisé, nous retrouvons l'en-tête YAML ainsi que quelques balises de templating, notamment dans la construction de la liste des billets :

```html
{% raw  %}
    {% for post in site.posts %}
	<li>
		<span>{{ post.date | date_to_string }}</span> 
		&raquo; <a href="{{ post.url }}">{{ post.title }}</a>
	</li>
	<p>{{ post.excerpt }}</p>
    {% endfor %}
{% endraw  %}
```

Le langage de templating est donc simple et lisible avec ici, une petite boucle `for`. Nous apercevons aussi un peu de la gestion des billets proposées par Jekyll.

A vrai dire, nous n'avons pas modifié grand chose dans ce fichier si ce n'est son titre.

L'édition d'un billet de blog était beaucoup plus interéssante. Le fichier fournit par défaut est écrit en Markdown et déjà, dans son nom, se trouvent des informations.

Jekyll interprète le nom d'un fichier de billet qui doit alors suivre la norme `YYYY-MM-DD-name-of-post.extension`.

