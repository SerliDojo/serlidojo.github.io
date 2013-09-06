---
layout: post
title:  "Frameworks Web #2 - Play! Framework"
date:   2012-11-29 19:27:13
comments: true
---

La série “Frameworks Web” des coding dojo à SERLI a connu son deuxième épisode. Cette soirée nous a permis de retrouver notre application SuperProsper et de la développer en Play! Framework. Ce dernier, plus qu'un simple framework, est une plate-forme Web complète (développement et déploiement) créée par Zenexity ([http://www.playframework.com](http://www.playframework.com)).


Le défi
==========

Le défi de la soirée était de réaliser une application de prospection de clients à partir de maquettes (page HTML et base de données SQL) et de quelques services fonctionnels, tout en respectant les pratiques couramment rencontrées en entreprise. Tour à tour, chacun a pris les commandes du PC pour construire petit à petit l'application, le tout guidé par le mentor de la soirée.


Un framework, une plate-forme
==========

Avant même de commencer à développer notre application, nous avons dû télécharger le kit de développement. En effet, Play! est livré avec un environnement en ligne de commandes permettant de créer, compiler et lancer les projets.

Nous avons donc utiliser la commande play new superprosper-play afin de créer et d'initialiser notre projet. Nous avions alors un nouveau dossier pour notre projet contenant le code de base d'une application de démonstration.

Juste après la création du projet, nous avons lancé la commande play run superprosper-play pour exécuter le projet. Play! lance alors un serveur pour déployer notre application et la compiler à la volée. Chaque modification d'un fichier source ou ressource entraîne ainsi la compilation et/ou la publication du projet.


Une classe Application, un routage automatique
==========

Le projet étant initialisé avec des classes de base, nous avons commencé par modifier la classe représentant l'application. Cette classe définit les points d'entrée de l'application, comme pour les applications Java classiques, sous la forme de méthodes publiques et statiques. Ainsi, la première méthode se nomme index et est associée, par défaut, à la racine de notre contexte Web. En entrant l'URL de notre application dans le navigateur, nous arrivons donc sur cette méthode.

Ce routage automatique est en fait opéré par un fichier de règles. En modifiant ce fichier, nous avons pu changer le nom de notre méthode. Cependant, une règle générique est présente et oriente les requêtes vers les classes et leurs méthodes en fonction de l'URL utilisée, selon les principes REST.

```
GET     /                                       Application.accueil

*       /{controller}/{action}                  {controller}.{action}
```

*Extrait du fichier routes*


Une page = un fichier HTML et une méthode statique
==========

Notre projet Play! dispose d'une page par défaut, associée à la méthode accueil de notre application. Par défaut, les pages HTML portent le nom de la méthode d'entrée et sont placées dans un dossier représentant la classe portant la méthode. La méthode render, appelée à la fin de notre méthode accueil, lance le rendu de la page HTML.

Ainsi, nous avons importé le code de notre maquette HTML dans cette page, en prenant soin d'adapter les références aux ressources statiques. L'application étant toujours en cours d'exécution, tout fut redéployé automatiquement. En accédant à la racine de contexte de l'application, la page principale s'afficha. L'application Play! "SuperProsper" était née.


Navigation naturelle selon routage
==========

Pour le moment l'application se résumait à une page statique. Nous avons donc décider de brancher le formulaire sur une nouvelle page, la page des résultats. Sur le même principe que la première, nous avons ajouter une nouvelle méthode statique, nommée cette fois recherche. Ensuite, nous avons importé le code HTML de la maquette dans la page correspondante.

Sans autre modification supplémentaire, la page était disponible à l'URL correspondante. Il ne nous restait plus qu'à la référencer comme action de notre formulaire pour connecter les deux pages.

```html
<form id="listFilters" class="form-inline" action="@{Application.recherche}">
	<select name="filtre">
		(...)
	</select> 
	<input class="span6" type="text" placeholder="Recherche" name="recherche" />
	<button type="submit" class="btn">Rechercher</button>
</form>
```

*Liaison vers une autre page Play!*


La liaison HTML - Java par paramètres
==========

Cependant, cette nouvelle page se devait d'accepter des paramètres correspondant à la saisie de l'utilisateur. Ces paramètres sont transmis simplement lors de la soumission du formulaire HTML. Nous avons donc ajouté deux paramètres à notre méthode recherche afin de récupérer les paramètres de la requête (paramètres dans l'URL en GET, paramètres dans le corps de la requête en  POST).

```java
public static void recherche(String filtre, String recherche) {
	(...)
}
```

*Récupération des paramètres de requête dans la classe Java*

Dans le sens inverse, les valeurs à transmettre à la page, comme le report des valeurs saisies dans les champs de recherche, se fait également via des paramètres d'appel. En effet, l'appel à la méthode render peut être accompagné de paramètres qui seront utilisables dans la page, simplement en utilisant le nom du paramètre.

```java
render(recherche);
```

*Transmission de valeurs à la vue*

```html
	<input class="span6" type="text" placeholder="Recherche"
			name="recherche" value="${recherche}"/>
```

*Utilisation de valeurs dans la vue*


Un composant = un fichier HTML
==========

Le formulaire de recherche étant commun aux deux pages, il nous a permis de voir comment factoriser une partie de la vue.

Dans cette méthode, nous accédons à la saisie de l'utilisateur par le biais du composant que nous avions référencé auparavant. En effet, le fait d'avoir ajouté le champ de saisie au formulaire, la valeur de ce dernier est envoyée avec la soumission du formulaire.

Une fois ce composant en place, il ne nous restait plus qu'à transmettre les données saisies dans le formulaire à la page de résultat. Cette page est elle-aussi implémentée par une classe Java et un fichier HTML. La navigation dans Wicket peut se faire de différentes manières, en indiquant la classe de la page suivante ainsi que les paramètres de la page (GET ou POST) ou simplement en instanciant cette classe Java. Il est alors possible de lui passer directement des valeurs. Nous avons choisi cette voie en passant les valeurs saisies dans le formulaire.

```java
Form<String> searchForm = new Form<String>("searchForm") {
	@Override
	protected void onSubmit() {
		(...)
		setResponsePage(new ResultPage(searchText, searchFilter));
	}
};
```
		
		
Un composant = un fichier HTML et une classe Java
==========

Cette manière de faire nous a paru simple mais quelque peu déroutante : étant toujours situé dans le constructeur de notre page, nous avions l'impression que tout notre code allait se retrouver dans ce seul constructeur avec de nombreuses classes internes.

Nous nous sommes alors empressés de chercher comment découpler ce code. L'une des solutions est apportée par Wicket dans son orientation composants. Nous avons pris cette direction et avons rassemblé tout le formulaire dans un panneau dédié.

Le code concernant la dynamique de ce panneau était alors regroupé dans une classe SearchPanel associée elle-aussi à un fichier HTML. La classe Java hérite de la classe Panel de Wicket et le code HTML inclut l'élément wicket:panel pour désigner le sous-ensemble HTML qui représente ce formulaire.

```html
<!DOCTYPE html>
<html>
<head>
...
</head>
<body>
	<wicket:panel>
	...
	</wicket:panel>
</body>
</html>
```

*Constitution d'un panneau Wicket HTML*

En tant que composant Wicket, notre panneau de recherche se devait de respecter les conventions Wicket et d'utiliser un objet de modèle. N'ayant pas de classe propre aux données du formulaire, nous avons utiliser une paire (couple du texte de recherche et du filtre).

```java
public class SearchPanel extends Panel {
	public SearchPanel(String id, IModel<Pair<String, Filtre>> model) {
		(...)
	}
}
```

*Déclaration d'un composant Wicket Java*

```java
Pair<String, Filtre> pair = new ImmutablePair<String, Filtre>(
		searchText, searchFilter);
add(new SearchPanel("searchPanel", Model.of(pair)));
```

*Appel du composant*


Suite et fin
==========

Le dojo s'est poursuivi par la découverte d'autres composants disponibles dans Wicket comme les listes déroulantes ou les tableaux. Nous avons également fait quelques essais avec les modèles plus complexes ou la gestion des requêtes AJAX.

Aussi, nous n'avons branché les services disponibles que très simplement, en les instanciant. Il existe des intégrations avec les moteurs d'injection classiques ainsi qu'avec d'autres frameworks comme Bean Validation mais nous avons concentré notre soirée sur la partie Web.
