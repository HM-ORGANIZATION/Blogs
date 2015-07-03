---
ID: 200
post_title: HawtIO, écrire un plugin
author: Gérald Quintana
post_date: 2014-01-14 10:30:00
post_excerpt: '<p>Après avoir découvert dans un <a href="http://blog.zenika.com/index.php?post/2014/01/07/HawtIO-la-console-web-polyvalente">premier article</a> les fonctionnalités de HawtIO, nous allons à présent voir comment développer son propre plugin.</p>'
layout: post
permalink: http://blog.zenika-offres.com/?p=200
published: true
slide_template:
  - default
---
Après avoir découvert dans un <a href="http://blog.zenika.com/index.php?post/2014/01/07/HawtIO-la-console-web-polyvalente">premier article</a> les fonctionnalités de HawtIO, nous allons à présent voir comment développer son propre plugin.

<!--more-->

L'exemple qui illustre cet article a pour objectif d'afficher dans un écran dédié de la console HawtIO, le contenu du registre JNDI d'un serveur d'application. Le code source de l'application est disponible dans le <a href="https://github.com/Zenika/Blogs/tree/master/20140101_HawtIO">GitHub Zenika</a>
<h3>Côté backend</h3>
<h2>Créer un MXBean</h2>
Pour créer un MBean JMX, il n'est nul besoin d'utiliser HawtIO, mais l'outil propose une classe de base <code>MBeanSupport</code> assez pratique. Elle se charge de l'enregistrement dans le serveur JMX. On écrira donc quelque chose du genre:
<pre class="java code java" style="font-family: inherit;"><span style="color: #008000; font-style: italic; font-weight: bold;">/** Interface */</span> @MXBean <span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">interface</span> JndiFacadeMXBean <span style="color: #009900;">{</span> 	... <span style="color: #009900;">}</span> <span style="color: #008000; font-style: italic; font-weight: bold;">/** Implémentation */</span> <span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> JndiFacade <span style="color: #000000; font-weight: bold;">extends</span> MBeanSupport <span style="color: #000000; font-weight: bold;">implements</span> JndiFacadeMXBean <span style="color: #009900;">{</span> 	... <span style="color: #009900;">}</span></pre>
Pour rappel, apparus avec JMX 2/Java 6, les MXBeans sont une extension et une simplification des MBeans JMX. Ils limitent les types autorisés dans la valeurs de retour et les paramètres mais permettent tout de même d'échanger des objets Java simples et augmentent les chances de compatibilité des applications clientes.

Pour en savoir plus:
<ul>
	<li><a href="https://weblogs.java.net/blog/emcmanus/archive/2006/02/what_is_an_mxbe.html">Qu'est un MXBean par le Spec lead de la JSR 255 JMX 2</a></li>
	<li><a href="http://docs.oracle.com/javase/tutorial/jmx/mbeans/mxbeans.html">Les MXBeans dans le tutoriel JMX</a></li>
	<li><a href="http://docs.oracle.com/javase/7/docs/api/javax/management/MXBean.html">Les MXBeans dans la JavaDoc</a></li>
</ul>
<h2>Apache Aries Blueprint</h2>
<a href="http://wiki.osgi.org/wiki/Blueprint">Blueprint</a> est une spécification d'un framework d'injection de dépendances adaptée aux conteneurs OSGi. Apache Aries est une implémentation de Blueprint ainsi qu'un ensemble de composants de base (JTA, JPA, JNDI...) pour OSGI et Blueprint. En clair, c'est une sorte de Spring, en différent; Blueprint est par exemple utilisé dans Apache ServiceMix. HawtIO peut s'appuyer sur Blueprint, même si ce n'est pas une obligation.

Pour découvrir Blueprint:
<ul>
	<li><a href="http://www.ibm.com/developerworks/opensource/library/os-osgiblueprint/index.html">Une présentation de Blueprint par IBM</a></li>
	<li><a href="http://aries.apache.org/modules/blueprint.html">Apache Aries Blueprint</a></li>
</ul>
Comme Spring, Blueprint se configure avec des fichiers XML (la syntaxe est quasi identique) et gère le cycle de vie des objets:
<pre class="xml code xml" style="font-family: inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;blueprint</span> <span style="color: #000066;">xmlns</span>=<span style="color: #ff0000;">"http://www.osgi.org/xmlns/blueprint/v1.0.0"</span></span> <span style="color: #009900;">		<span style="color: #000066;">xmlns:ext</span>=<span style="color: #ff0000;">"http://aries.apache.org/blueprint/xmlns/blueprint-ext/v1.2.0"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span> 	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;bean</span> <span style="color: #000066;">id</span>=<span style="color: #ff0000;">"javaJndiFacade"</span> <span style="color: #000066;">class</span>=<span style="color: #ff0000;">"io.hawt.jndi.JndiFacade"</span> </span> <span style="color: #009900;">			<span style="color: #000066;">init-method</span>=<span style="color: #ff0000;">"init"</span> <span style="color: #000066;">destroy-method</span>=<span style="color: #ff0000;">"destroy"</span> <span style="color: #000066;">scope</span>=<span style="color: #ff0000;">"singleton"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span> 		<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"contextName"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"java:"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span> 	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/bean<span style="color: #000000; font-weight: bold;">&gt;</span></span></span> <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/blueprint<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
La méthode <code>init</code> se charge d'enregistrer le MBean dans la registre JMX, et inversement avec la méthode <code>destroy</code>
<h2>La découverte de plugins</h2>
Côté backend, BluePrint permet la modularité d'HawtIO. Au démarrage de l'application HawtIO, BluePrint parcourt les Jars présents dans le classpath à la recherche de fichiers <code>OSGI-INF/blueprint/blueprint.xml</code>, puis instancie et configure les beans (des MXBean par exemple) comme tout conteneur d'injection de dépendances qui se respecte. En ajoutant un Jar sur le classpath d'HawtIO, on peut donc faire apparaître des MBeans dans le registre JMX.

Coté frontend, pour découvrir dynamiquement les plugins additionnels, HawtIO recherche dans le registre JMX, les MBeans de type <code>plugin</code> dans le domaine <code>hawtio</code>. Pour cette raison, on déclare un second MBean dans Blueprint comme précédemment, à ceci près que l'implémentation est fournie par HawtIO cette fois:
<pre class="xml code xml" style="font-family: inherit;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;bean</span> <span style="color: #000066;">id</span>=<span style="color: #ff0000;">"jndiPlugin"</span> <span style="color: #000066;">class</span>=<span style="color: #ff0000;">"io.hawt.web.plugin.HawtioPlugin"</span> </span> <span style="color: #009900;">			<span style="color: #000066;">init-method</span>=<span style="color: #ff0000;">"init"</span> <span style="color: #000066;">destroy-method</span>=<span style="color: #ff0000;">"destroy"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span> 		<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"name"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"jndi"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span> 		<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"context"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"/hawtio"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span> 		<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"scripts"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"app/jndi/js/jndiPlugin.js,app/jndi/js/jndiController.js"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span> 	<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/bean<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
On obtient ainsi un MBean nommé <code>hawtio:type=plugin,name=jndi</code>. La propriété <code>scripts</code> donne la liste de fichiers JavaScript que devra charger dynamiquement le navigateur.
<h3>Côté frontend</h3>
C'est de l'AngularJS pur jus: je ne détaillerai guère l'utilisation de ce framework.
<h2>Initialisation du plugin</h2>
On créé un module AngularJS dans lequel on déclare des routes. Lorsque le module démarre (fonction <code>run</code>), on inscrit dans des registres:
<ul>
	<li>la documentation (<code>addUserDoc</code>),</li>
	<li>les onglets de premier niveau (<code>topLevelTabs</code>),</li>
	<li>les onglets dans la vue JMX (<code>subLevelTabs</code></li>
	<li>et le module lui même (<code>addModule</code>).</li>
</ul>
<img style="display: block; margin: 0 auto;" title="HawtIO Top level Tabs" src="/wp-content/uploads/2015/07/hawtio-plugin-topLevelTabs2.png" alt="User doc, Top level tabs, Layout" />

<img style="display: block; margin: 0 auto;" title="HawtIO Sub level Tabs" src="/wp-content/uploads/2015/07/hawtio-plugin-subLevelTabs2.png" alt="Sub level tabs" />

Au niveau du code, ça donne:
<pre class="javascript code javascript" style="font-family: inherit;">angular.<span style="color: #660066;">module</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'Jndi'</span><span style="color: #339933;">,</span> <span style="color: #009900;">[</span><span style="color: #3366cc;">'hawtioCore'</span><span style="color: #009900;">]</span><span style="color: #009900;">)</span> 		.<span style="color: #660066;">config</span><span style="color: #009900;">(</span><span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>$routeProvider<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 			$routeProvider 					.<span style="color: #660066;">when</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'/jndi'</span><span style="color: #339933;">,</span> <span style="color: #009900;">{</span>templateUrl<span style="color: #339933;">:</span> <span style="color: #3366cc;">'app/jndi/html/jndi.html'</span><span style="color: #009900;">}</span><span style="color: #009900;">)</span> 					.<span style="color: #660066;">when</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'/jndi/:name'</span><span style="color: #339933;">,</span> <span style="color: #009900;">{</span>templateUrl<span style="color: #339933;">:</span> <span style="color: #3366cc;">'app/jndi/html/jndi.html'</span><span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 		<span style="color: #009900;">}</span><span style="color: #009900;">)</span> 		.<span style="color: #660066;">run</span><span style="color: #009900;">(</span><span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>workspace<span style="color: #339933;">,</span> viewRegistry<span style="color: #339933;">,</span> helpRegistry<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 			viewRegistry<span style="color: #009900;">[</span><span style="color: #3366cc;">"jndi"</span><span style="color: #009900;">]</span> <span style="color: #339933;">=</span> <span style="color: #3366cc;">"app/jndi/html/layoutJndiTabs.html"</span><span style="color: #339933;">;</span> 			<span style="color: #006600; font-style: italic;">// Documentation</span> 			helpRegistry.<span style="color: #660066;">addUserDoc</span><span style="color: #009900;">(</span><span style="color: #3366cc;">"jndi"</span><span style="color: #339933;">,</span> <span style="color: #3366cc;">'app/jndi/doc/help.md'</span><span style="color: #339933;">,</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span><span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 				<span style="color: #000066; font-weight: bold;">return</span> workspace.<span style="color: #660066;">treeContainsDomainAndProperties</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'hawtio'</span><span style="color: #339933;">,</span> <span style="color: #009900;">{</span>type<span style="color: #339933;">:</span> <span style="color: #3366cc;">'JndiFacade'</span><span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 			<span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 			<span style="color: #006600; font-style: italic;">// Onglet premier niveau</span> 			workspace.<span style="color: #660066;">topLevelTabs</span>.<span style="color: #660066;">push</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span> 				id<span style="color: #339933;">:</span> <span style="color: #3366cc;">"jndi"</span><span style="color: #339933;">,</span> 				content<span style="color: #339933;">:</span> <span style="color: #3366cc;">"JNDI"</span><span style="color: #339933;">,</span> 				title<span style="color: #339933;">:</span> <span style="color: #3366cc;">"Browse JNDI registry"</span><span style="color: #339933;">,</span> 				isValid<span style="color: #339933;">:</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>workspace<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 					<span style="color: #000066; font-weight: bold;">return</span> workspace.<span style="color: #660066;">treeContainsDomainAndProperties</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'hawtio'</span><span style="color: #339933;">,</span> <span style="color: #009900;">{</span>type<span style="color: #339933;">:</span> <span style="color: #3366cc;">'JndiFacade'</span><span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 				<span style="color: #009900;">}</span><span style="color: #339933;">,</span> 				href<span style="color: #339933;">:</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span><span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 					<span style="color: #000066; font-weight: bold;">return</span> <span style="color: #3366cc;">"#/jndi"</span><span style="color: #339933;">;</span> 				<span style="color: #009900;">}</span> 			<span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 			<span style="color: #006600; font-style: italic;">// Onglet niveau JMX</span> 			workspace.<span style="color: #660066;">subLevelTabs</span>.<span style="color: #660066;">push</span><span style="color: #009900;">(</span><span style="color: #009900;">{</span> 				content<span style="color: #339933;">:</span> <span style="color: #3366cc;">'&lt;i class="icon-list-alt"&gt;&lt;/i&gt; Java:'</span><span style="color: #339933;">,</span> 				title<span style="color: #339933;">:</span> <span style="color: #3366cc;">"Java: Context"</span><span style="color: #339933;">,</span> 				isValid<span style="color: #339933;">:</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>workspace<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 					<span style="color: #000066; font-weight: bold;">return</span> workspace.<span style="color: #660066;">hasDomainAndProperties</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'hawtio'</span><span style="color: #339933;">,</span> <span style="color: #009900;">{</span>type<span style="color: #339933;">:</span> <span style="color: #3366cc;">'JndiFacade'</span><span style="color: #339933;">,</span> <span style="color: #000066;">name</span><span style="color: #339933;">:</span> <span style="color: #3366cc;">"java"</span><span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 				<span style="color: #009900;">}</span><span style="color: #339933;">,</span> 				href<span style="color: #339933;">:</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span><span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 					<span style="color: #000066; font-weight: bold;">return</span> <span style="color: #3366cc;">"#/jndi/java"</span><span style="color: #339933;">;</span> 				<span style="color: #009900;">}</span> 			<span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 		<span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> hawtioPluginLoader.<span style="color: #660066;">addModule</span><span style="color: #009900;">(</span><span style="color: #3366cc;">'Jndi'</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span></pre>
Vous remarquerez au passage que les onglets apparaissent ou pas en fonction de la présence ou pas d'un MBean donné: <code>workspace.hasDomainAndProperties</code>. HawtIO va assez loin dans la dynamicité: par exemple, l'arrêt d'une application provoque le dés-enregistrement de MBeans donnés côté backend, aussitôt les onglets associés vont disparaître dans le frontend.
<h2>Layout</h2>
Dans l'initialisation ci-dessus, on a déclaré dans le <code>viewRegistry</code> un layout, celui-ci donne la mise en page globale de l'écran.
<pre> &lt;ul class="nav nav-tabs" ng-controller="Core.NavBarController"&gt; 	&lt;!-- Onglet niveau layout --&gt; 	&lt;li ng-class='{active : isActive("#/jndi/java")}'&gt; 		&lt;a ng-href="{{link('#/jndi/java')}}"&gt;Java:&lt;/a&gt; 	&lt;/li&gt; &lt;/ul&gt; &lt;div class="row-fluid"&gt; 	&lt;div ng-view&gt;&lt;/div&gt; &lt;/div&gt;</pre>
Contrairement aux autres, les onglets du niveau layout sont décrits en HTML.
<h2>Vue, Contrôleur et Documentation</h2>
Ici aussi c'est de l'AngularJS classique, je vous épargnerai donc les détails:
<pre class="javascript code javascript" style="font-family: inherit;">Jndi.<span style="color: #660066;">JndiController</span> <span style="color: #339933;">=</span> <span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>$scope<span style="color: #339933;">,</span> $routeParams<span style="color: #339933;">,</span> jolokia<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 	$scope.<span style="color: #660066;">context</span> <span style="color: #339933;">=</span> <span style="color: #009900;">{</span>id<span style="color: #339933;">:</span> $routeParams.<span style="color: #000066;">name</span><span style="color: #009900;">}</span><span style="color: #339933;">;</span> <span style="color: #009900;">}</span></pre>
<pre> &lt;div ng-controller="Jndi.JndiController"&gt; 	{{context.id}} &lt;/div&gt;</pre>
Dans le contrôleur, on utilise l'<a href="http://www.jolokia.org/reference/html/clients.html#client-javascript">API JavaScript Jolokia</a> pour aller chercher des informations sur le backend.
<pre class="javascript code javascript" style="font-family: inherit;">jolokia.<span style="color: #660066;">getAttribute</span><span style="color: #009900;">(</span><span style="color: #3366cc;">"hawtio:type=JndiFacade,name=java"</span><span style="color: #339933;">,</span><span style="color: #3366cc;">"ContextName"</span><span style="color: #339933;">,</span><span style="color: #009900;">{</span> 		method<span style="color: #339933;">:</span><span style="color: #3366cc;">"POST"</span><span style="color: #339933;">,</span> 		success<span style="color: #339933;">:</span><span style="color: #003366; font-weight: bold;">function</span><span style="color: #009900;">(</span>response<span style="color: #009900;">)</span> <span style="color: #009900;">{</span> 			$scope.<span style="color: #660066;">context</span>.<span style="color: #000066;">name</span><span style="color: #339933;">=</span>response<span style="color: #339933;">;</span> 			$scope.$apply<span style="color: #009900;">(</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span> 		<span style="color: #009900;">}</span> 	<span style="color: #009900;">}</span><span style="color: #009900;">)</span><span style="color: #339933;">;</span></pre>
Vous noterez, que comme Jolokia JS ne s'appuie pas sur le <code>$http</code> d'Angular mais sur JQuery, on est contraint d'invoquer <code>$apply</code> dans les callbacks pour que la vue soit mise à jour.

En ce qui concerne l'aide en ligne du plugin, c'est un simple fichier écrit en MarkDown.
<h3>Conclusion</h3>
Le développement de plugins HawtIO n'est pas sorcier, l'outil est vraiment conçu avec l'extensibilité en ligne de mire. On regrettera cependant que la documentation soit pauvre (pas JSDoc pour l'instant), il faudra donc disséquer un peu le code source.

Quelques pointeurs:
<ul>
	<li><a href="http://hawt.io/plugins/howPluginsWork.html">Qu'est-ce qu'un plugin HawtIO?</a></li>
	<li><a href="https://github.com/hawtio/hawtio/blob/master/DEVELOPERS.md">Développer dans HawtIO</a></li>
</ul>