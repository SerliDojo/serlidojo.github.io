---
layout: post
title:  "Frameworks Web #1 - Wicket"
date:   2012-04-05 19:27:13
comments: true
---

Dernièrement, dans les locaux de SERLI, s'est déroulé le premier coding dojo de la série “Frameworks Web”. Cette soirée fut l'occasion pour plusieurs d'entre nous de découvrir Wicket, un framework orienté composant, de la communauté Apache [http://wicket.apache.org].


Le défi
==========


Le défi de la soirée était de réaliser une application de prospection de clients à partir de maquettes (page HTML et base de données SQL) et de quelques services fonctionnels, tout en respectant les pratiques couramment rencontrées en entreprise. Tour à tour, des binômes se sont succédés devant le PC pour construire petit à petit l'application. Ceux d'entre nous qui connaissaient déjà un peu Wicket aiguillaient les novices dans leurs choix et leurs réflexions au moment de coder.

Un projet Maven, une classe Application
==========

Nous avons tout d'abord commencé par créer un projet Maven avec une dépendance sur Wicket (forcément !). Nous avons ensuite créé une classe symbolisant une application Wicket et référencé cette classe dans le web.xml du projet avec le filtre Wicket. Cette classe permet de définir la configuration Wicket de manière globale à l'application, comme par exemple la classe à utiliser pour rendre la page d'accueil.

```xml
<filter>
	<filter-name>superprosper</filter-name>
	<filter-class>org.apache.wicket.protocol.http.WicketFilter</filter-class>
	<init-param>
		<param-name>applicationClassName</param-name>
		<param-value>
			com.serli.dojo.superprosper.wicket.SuperProsperApplication
		</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>superprosper</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```
*Extrait du fichier web.xml*

```java
public class SuperProsperApplication extends WebApplication {
	@Override
	public Class<? extends Page> getHomePage() {
		return HomePage.class;
	}
}
```
*Extrait du fichier SuperProsperApplication.java*


Une page = un fichier HTML et une classe Java
==========

La création de la première classe de rendu, représentant une page Wicket, fut l'occasion de voir la première spécificité de ce framework, à savoir le couplage entre une page HTML et une classe Java, nommées à l'identique et située côte à côté dans le classpath. Cette première étape nous a permis de voir que Wicket appliquait le modèle MVC et forçait l'utilisateur à le suivre également. En effet, la partie "Vue" est assurée par cette association entre la classe Java et le fichier HTML.

Nous avons alors recopié la page HTML de la maquette dans la page HTML précédemment créée, nous avons déployé notre application et accédé à la racine de contexte de l'application. La page principale s'afficha, l'application Wicket "SuperProsper" était née.

![Première page]({{ site.url }}/img/maquette.png)


La liaison HTML ↔ Java par identifiant Wicket
==========

Pour le moment l'application se résumait à une page statique. Nous avons donc continué en remplaçant les composants statiques de la page HTML par des composants Wicket instanciés directement dans la classe Java de la page. Nous avons alors découvert l'attribut wicket:id à placer dans les tags HTML que l'on veut traiter en Wicket, via la classe Java. La valeur de cet attribut est à passer au moment de l'appel du constructeur du composant Wicket.

```html
<input wicket:id="searchText" type="text" placeholder="Recherche" />
```
*Déclaration d'un identifiant Wicket pour le composant HTML*

```java
TextField<String> searchTextFiled = new TextField<String> (
		"searchText", new Model<String>(""));
```
*Déclaration d'un composant Wicket dans la classe Java*

Pour débuter, nous nous sommes contentés de lier le champ de saisie de la recherche ainsi que le formulaire HTML qui englobe ce champ. Le bouton associé étant un bouton HTML de type submit, il n'était pas nécessaire de créer une liaison.

Le formulaire était le candidat idéal pour illustrer comment Wicket traite le retour utilisateur et, de manière plus général, comment Wicket nous propose de personnaliser les composants. En effet, nous avons développé une classe anonyme surchargeant la méthode onSubmit du formulaire.

Dans cette méthode, nous accédons à la saisie de l'utilisateur par le biais du composant que nous avions référencé auparavant. En effet, le fait d'avoir ajouté le champ de saisie au formulaire, la valeur de ce dernier est envoyée avec la soumission du formulaire.


Les données englobées dans un objet Wicket
==========

Wicket passe par un système de modèle pour séparer les données métier des composants d'affichage. A la création d'un composant, le développeur doit renseigner un modèle. Cet objet encapsule la donnée afin de pouvoir redéfinir son chargement et la lier au composant graphique. Nous récupérons donc la valeur saisie dans le champ via ce modèle.

```java
Form<String> searchForm = new Form<String>("searchForm") {
	@Override
	protected void onSubmit() {
		String searchText = searchTextField.getModelObject()
		(...)
	}
};
searchForm.add(searchText);
```
*Personnalisation d'un composant formulaire HTML en Wicket*


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


Navigation par les classes Java
==========

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
*Appel d'une nouvelle page Wicket*


Suite et fin
==========

Le dojo s'est poursuivi par la découverte d'autres composants disponibles dans Wicket comme les listes déroulantes ou les tableaux. Nous avons également fait quelques essais avec les modèles plus complexes ou la gestion des requêtes AJAX.

Aussi, nous n'avons branché les services disponibles que très simplement, en les instanciant. Il existe des intégrations avec les moteurs d'injection classiques ainsi qu'avec d'autres frameworks comme Bean Validation mais nous avons concentré notre soirée sur la partie Web.
