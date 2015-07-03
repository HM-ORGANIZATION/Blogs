---
ID: 107
post_title: Introduction aux DSL en Groovy
author: ggiamarchi
post_date: 2012-02-27 12:08:00
post_excerpt: '<p>Si vous ne connaissez pas encore <a href="/index.php?tag/groovy">Groovy</a>, voici une bonne occasion de vous y mettre. Au travers d’un exemple, je vais vous présenter plusieurs des concepts offerts par le langage qui vont nous permettre de créer assez simplement un DSL.</p>'
layout: post
permalink: http://blog.zenika-offres.com/?p=107
published: true
---
<p>Si vous ne connaissez pas encore <a href="/index.php?tag/groovy">Groovy</a>, voici une bonne occasion de vous y mettre. Au travers d’un exemple, je vais vous présenter plusieurs des concepts offerts par le langage qui vont nous permettre de créer assez simplement un DSL.</p>
<!--more-->
<h3>Qu’est-ce qu’un DSL&nbsp;?</h3> <p>Le terme DSL (Domain Specific Language) est apparu il y a quelques années. Il désigne un langage dont les spécifications - et en premier lieu la syntaxe - sont spécifiquement adaptées à un domaine fonctionnel donné contrairement aux langages de programmation généralistes. Un certains nombre de langages que nous connaissons bien sont des DSL même si nous ne sommes pas forcément habitués à les désigner comme tels. Par exemple, HTML est conçu pour la description de pages Web, les expressions régulières sont destinées à décrire des motifs de chaînes de caractères et le SQL permet d'exprimer des requêtes vers une base de données. Martin Fowler donne une bonne définition des DSL dans <a href="http://martinfowler.com/bliki/DomainSpecificLanguage.html">un petit article</a> sur son blog.</p> <p>Le langage Groovy offre des possibilités très avancées dans ce domaine. Les produits les plus connus du monde Groovy définissent souvent des DSL spécifiques à leurs besoins. C’est notamment le cas du framework <a href="/index.php?tag/grails">Grails</a> et de l’outil de build <a href="/index.php?tag/gradle">Gradle</a>.</p> <h3>La problématique</h3> <p>Nous allons créer un DSL adapté à la description d'un email. Une fois le DSL défini, il s'agira de l'interfacer avec une API d'envoi d’emails, en l'occurrence celle de Spring. L'objectif est de pouvoir envoyer des emails depuis notre code Groovy à l'aide d'une syntaxe intuitive et adaptée à ce besoin.</p> <p>Voici un exemple de code que je souhaite écrire pour envoyer un email.</p> <pre class="groovy code groovy" style="font-family:inherit">Email.<span style="color: #006600;">send</span> <span style="color: #66cc66;">&#123;</span> 	from <span style="color: #ff0000;">&quot;foo.bar@zenika.com&quot;</span> 	to <span style="color: #ff0000;">&quot;toto@zenika.com&quot;</span> 	cc <span style="color: #ff0000;">&quot;titi@zenika.com&quot;</span> 	cc <span style="color: #ff0000;">&quot;tutu@zenika.com&quot;</span> 	bcc <span style="color: #ff0000;">&quot;tata@zenika.com&quot;</span> 	subject <span style="color: #ff0000;">&quot;Ouaiiis !!! un mail envoyé via un DSL en Groovy&quot;</span> 	body <span style="color: #ff0000;">&quot;Pas grand chose à dire...&quot;</span> 	attach <span style="color: #ff0000;">&quot;file:/E:/file1.txt&quot;</span> 	attach <span style="color: #ff0000;">&quot;file:/E:/file2.txt&quot;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Il peut y avoir plusieurs lignes <code>to</code>, <code>cc</code> et <code>bcc</code>&nbsp;; une par destinataire. Il en va de même pour les pièces jointes, une ligne par fichier. L'ordre des lignes est sans importance. Par exemple la ligne <code>from</code> pourrait tout à fait se trouver en dernier et plusieurs lignes <code>to</code> ne sont pas nécessairement adjacentes.</p> <p>Le bloc de code se trouvant à l'intérieur de la paire d'acolades est une closure Groovy.</p> <h3>Un aperçu des closures</h3> <p>Avant de rentrer dans le vif du sujet, je souhaite introduire les closures pour ceux qui ne seraient pas familiarisés avec ce concept. Pour faire simple, il s'agit d'un bloc de code qui est déclaré pour une exécution ultérieure. Ce bloc de code peut être assigné à une variable afin de pouvoir être référencé par la suite.</p> <p>On peut voir cela comme une sorte de pointeur sur une fonction mais avec un comportement plus proche d'une classe interne que d'une méthode, notamment au niveau de l'accès aux variables externes. Par exemple, le code d'une closure peut utiliser toutes les variables accessibles depuis de contexte de déclaration de la closure. En terme d'écriture de code, une classe anonyme Java est ce qui se rapproche le plus d'une closure. Elle peut également accéder aux variables externes avec par contre une restriction, les variables doivent avoir été déclarées <code>final</code>.</p> <p>Une closure peut prendre zéro, un seul ou plusieurs paramètres.</p> <p>Comme un exemple clair vaut mieux que de longues explications, voici la déclaration d'une closure à un paramètre en Groovy.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">def</span> addTva <span style="color: #66cc66;">=</span> <span style="color: #66cc66;">&#123;</span> prixHt <span style="color: #66cc66;">-&gt;</span> 	prixHt <span style="color: #66cc66;">*</span> <span style="color: #cc66cc;">1.196</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Un appel à cette closure peut être écrit de deux manières différentes</p> <ul> <li>avec la même syntaxe que pour un appel de méthode</li> <li>en invoquant la méthode <code>call(Object...)</code> de la classe <code>groovy.lang.Closure</code></li> </ul> <pre class="groovy code groovy" style="font-family:inherit">prixTtc <span style="color: #66cc66;">=</span> addTva<span style="color: #66cc66;">&#40;</span><span style="color: #cc66cc;">20</span><span style="color: #66cc66;">&#41;</span> prixTtc <span style="color: #66cc66;">=</span> addTva.<span style="color: #993399; font-weight: bold;">call</span><span style="color: #66cc66;">&#40;</span><span style="color: #cc66cc;">20</span><span style="color: #66cc66;">&#41;</span></pre> <p>Nous allons plus tard nous apercevoir que ce mécanisme sera le point central du DSL que nous allons construire.</p> <h3>Implémentation du DSL</h3> <p>Il existe plusieurs façons d'implémenter un DSL en groovy, en utilisant les caractérictiques dynamiques du langage ou encore en opérant des transformations AST (abstract syntax tree). Nous allons voir ici une méthode relativement simple qui se rapproche du pattern Builder en Java dans le sens ou nous allons créer un objet qui va stocker des informations grâce à des appels de méthodes successifs pour ensuite produire un résultat.</p> <p>A titre de comparaison, voici à quoi ressemblerait l'utilisation d'un builder en Java.</p> <pre class="java code java" style="font-family:inherit"><span style="color: #7F0055; font-weight: bold;">new</span> EmailBuilder<span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span> 	.<span style="color: #000000;">from</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;foo.bar@zenika.com&quot;</span><span style="color: #000000;">&#41;</span> 	.<span style="color: #000000;">to</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;toto@zenika.com&quot;</span><span style="color: #000000;">&#41;</span> 	.<span style="color: #000000;">subject</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Ouaiiis !!! Un mail envoyé via un DSL en Groovy&quot;</span><span style="color: #000000;">&#41;</span> 	.<span style="color: #000000;">body</span><span style="color: #000000;">&#40;</span><span style="color: #888888;">&quot;Pas grand chose à dire...&quot;</span><span style="color: #000000;">&#41;</span> 	.<span style="color: #000000;">send</span><span style="color: #000000;">&#40;</span><span style="color: #000000;">&#41;</span>;</pre> <p>Afin de découpler notre DSL de l’implementation du gestionnaire d’envoi d’emails que nous allons écrire ensuite, commençons par définir une interface qui déclare une unique méthode permettant d'envoyer un email. Nous fournirons plus tard une implémentation de cette interface.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">interface</span> EmailSender <span style="color: #66cc66;">&#123;</span> 	<span style="color: #000000; font-weight: bold;">def</span> send<span style="color: #66cc66;">&#40;</span>Email email<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#125;</span> &nbsp; <span style="color: #000000; font-weight: bold;">class</span> Email <span style="color: #66cc66;">&#123;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Nous pouvons à présent passer aux choses sérieuses et nous lancer dans l'implémentation du DSL.</p> <p>Nous allons créer une classe <code>Email</code
> qui contient&nbsp;:</p> <ul> <li>Pour chacun des mots-clés qui constituent notre langage une méthode du même nom (from, to, cc, bcc, subject, body, attach) qui prend une chaîne de caractères en paramètre</li> <li>Un attribut de type <code>EmailSender</code> (dans lequel une instance de la classe d'implémentation sera plus tard injectée par Spring)</li> <li>Une méthode statique <code>send</code> qui sera le point d'entrée de notre DSL</li> </ul> <p>Chacune des méthodes du DSL va se contenter de stocker dans un champ de la classe la valeur reçue en paramètre. Ainsi, on pourra par la suite passer en paramètre de la méthode <code>EmailSender.send(Email)</code> la référence de l'instance de la classe <code>Email</code> qui contiendra toutes les informations.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">class</span> Email <span style="color: #66cc66;">&#123;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #aaaadd; font-weight: bold;">String</span> from 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">def</span> to <span style="color: #66cc66;">=</span> <span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span> 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">def</span> cc <span style="color: #66cc66;">=</span> <span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span> 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">def</span> bcc <span style="color: #66cc66;">=</span> <span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span> 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #aaaadd; font-weight: bold;">String</span> subject 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #aaaadd; font-weight: bold;">String</span> body 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">def</span> attachedFile <span style="color: #66cc66;">=</span> <span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">static</span> EmailSender sender &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> from<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> from<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		<span style="color: #000000; font-weight: bold;">this</span>.<span style="color: #006600;">from</span> <span style="color: #66cc66;">=</span> from 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> to<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> recipient<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		to <span style="color: #66cc66;">&lt;&lt;</span> recipient 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> cc<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> recipient<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		cc <span style="color: #66cc66;">&lt;&lt;</span> recipient 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> bcc<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> recipient<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		bcc <span style="color: #66cc66;">&lt;&lt;</span> recipient 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> subject<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> subject<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		<span style="color: #000000; font-weight: bold;">this</span>.<span style="color: #006600;">subject</span> <span style="color: #66cc66;">=</span> subject 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> body<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> body<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		<span style="color: #000000; font-weight: bold;">this</span>.<span style="color: #006600;">body</span> <span style="color: #66cc66;">=</span> body 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> attach<span style="color: #66cc66;">&#40;</span><span style="color: #aaaadd; font-weight: bold;">String</span> uri<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		attachedFile <span style="color: #66cc66;">&lt;&lt;</span> uri 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> send<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		sender.<span style="color: #006600;">send</span><span style="color: #66cc66;">&#40;</span><span style="color: #000000; font-weight: bold;">this</span><span style="color: #66cc66;">&#41;</span> 	<span style="color: #66cc66;">&#125;</span> &nbsp; <span style="color: #66cc66;">&#125;</span></pre> <p>Pour chaque envoi d'email on va construire une instance de la classe <code>Email</code> et chacune des instructions présentes dans la closure va devenir un appel de méthode sur cette instance. Pour cela, on utilise la caractéristique de délégation des closures.</p> <p>La classe <code>Closure</code> dispose d'un champ <code>delegate</code> qui contient la référence de l'objet sur lequel vont s'appliquer les appels de méthodes fait à l'intérieur de la closure. Par défaut, cet attribut est valorisé avec la référence de l'instance qui a exécuté la déclaration de la closure. Sa valeur peut être modifiée et c'est précisément ce que nous allons faire ici avant d'appeler la closure en lui assigant la référence de l'objet de type <code>Email</code> nouvellement créé.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">def</span> <span style="color: #000000; font-weight: bold;">static</span> send<span style="color: #66cc66;">&#40;</span>Closure closure<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 	Email email <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> Email<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> 	closure.<span style="color: #006600;">delegate</span> <span style="color: #66cc66;">=</span> email       <span style="color: #808080; font-style: italic;">// Mise en place de la délégation à l'objet email </span> 	closure.<span style="color: #993399; font-weight: bold;">call</span><span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span>                     <span style="color: #808080; font-style: italic;">// Appel de la closure</span> 	email.<span style="color: #006600;">send</span><span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Rendons le constructeur privé car il n'y a aucune raison d'instancier la classe <code>Email</code> en dehors de la méthode <code>send(Closure)</code></p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-w
eight: bold;">private</span> Email<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> &nbsp; <span style="color: #66cc66;">&#125;</span></pre> <p>Ecrivons à présent un stub de l'interface EmailSender afin de pourvoir d'ores et déjà tester notre DSL.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">class</span> EmailSenderStub <span style="color: #000000; font-weight: bold;">implements</span> EmailSender <span style="color: #66cc66;">&#123;</span> 	<span style="color: #000000; font-weight: bold;">def</span> send<span style="color: #66cc66;">&#40;</span>Email email<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		<span style="color: #993399;">println</span> email 	<span style="color: #66cc66;">&#125;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Etant donnée l'implémentation du stub, il est évidement souhaitable d'implémenter la méthode <code>toString()</code> sur le classe <code>Email</code>.</p> <pre class="groovy code groovy" style="font-family:inherit">@Override <span style="color: #000000; font-weight: bold;">public</span> <span style="color: #aaaadd; font-weight: bold;">String</span> toString<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span>         <span style="color: #ff0000;">&quot;&quot;&quot;       from     : ${from}       to       : ${to}       cc       : ${cc}       bcc      : ${bcc}       subject  : ${subject}       body     : ${body}       attached : ${attachedFile}      &quot;&quot;&quot;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Injectons une instance du stub dans l'attribut <code>Email.sender</code>.</p> <pre class="groovy code groovy" style="font-family:inherit">Email.<span style="color: #006600;">sender</span> <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> EmailSenderStub<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span></pre> <p>Nous sommes maintenant prêt à tester notre DSL</p> <pre class="groovy code groovy" style="font-family:inherit">Email.<span style="color: #006600;">send</span> <span style="color: #66cc66;">&#123;</span> 	from <span style="color: #ff0000;">&quot;foo.bar@zenika.com&quot;</span> 	to <span style="color: #ff0000;">&quot;toto@zenika.com&quot;</span> 	cc <span style="color: #ff0000;">&quot;titi@zenika.com&quot;</span> 	cc <span style="color: #ff0000;">&quot;tutu@zenika.com&quot;</span> 	bcc <span style="color: #ff0000;">&quot;tata@zenika.com&quot;</span> 	subject <span style="color: #ff0000;">&quot;Ouaiiis !!! un mail envoyé via un DSL en Groovy&quot;</span> 	body <span style="color: #ff0000;">&quot;Pas grand chose à dire...&quot;</span> 	attach <span style="color: #ff0000;">&quot;file:/E:/file1.txt&quot;</span> 	attach <span style="color: #ff0000;">&quot;file:/E:/file2.txt&quot;</span> <span style="color: #66cc66;">&#125;</span></pre> <p>Il est à noter ici que l’ordre dans lequel les champs sont renseignés lors de la construction de l’email est sans importance puisque chaque appel de méthode va uniquement permettre de stocker une nouvelle information. Et ce n’est qu’une fois que la closure aura terminé sont exécution que l’appel à la méthode <code>send()</code> sera effectué.</p> <p>On peut voir que la closure passée en paramètre de la méthode statique <code>Email.send(Closure)</code> est une succession d'appels aux différentes méthodes déclarées sur la classe <code>Email</code>, d'où la nécessité de déléguer à celle-ci les appels de méthodes fait à l’intérieur de la closure.</p> <p>La syntaxe de Groovy, plus souple que celle de Java nous autorise à supprimer les parenthèses des appels de méthodes et les points-virgules en fin d'instruction. Grâce à cela, la syntaxe du DSL se rapproche plus du langage naturel. A tritre de comparaison, voici le même code écrit dans une syntaxe à la façon Java.</p> <pre class="groovy code groovy" style="font-family:inherit">Email.<span style="color: #006600;">send</span> <span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#123;</span> 	from<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;foo.bar@zenika.com&quot;</span><span style="color: #66cc66;">&#41;</span>; 	to<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;toto@zenika.com&quot;</span><span style="color: #66cc66;">&#41;</span>; 	cc<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;titi@zenika.com&quot;</span><span style="color: #66cc66;">&#41;</span>; 	cc<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;tutu@zenika.com&quot;</span><span style="color: #66cc66;">&#41;</span>; 	bcc<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;tata@zenika.com&quot;</span><span style="color: #66cc66;">&#41;</span>; 	subject<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;Ouaiiis !!! un mail envoyé via un DSL en Groovy&quot;</span><span style="color: #66cc66;">&#41;</span>; 	body<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;Pas grand chose à dire...&quot;</span><span style="color: #66cc66;">&#41;</span>; 	attach<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;file:/E:/file1.txt&quot;</span><span style="color: #66cc66;">&#41;</span>; 	attach<span style="color: #66cc66;">&#40;</span><span style="color: #ff0000;">&quot;file:/E:/file2.txt&quot;</span><span style="color: #66cc66;">&#41;</span>; <span style="color: #66cc66;">&#125;</span><span style="color: #66cc66;">&#41;</span>;</pre> <p>En terme d'exécution ces deux syntaxes sont strictement équivalentes.</p> <h3>Implémentation du gestionnaire d’envoi d’emails</h3> <p>Voici une implémentation simple de l’interface <code>EmailSender</code> utilisant l'API de Spring.</p> <pre class="groovy code groovy" style="font-family:inherit"><span style="color: #000000; font-weight: bold;">import</span> <span style="color: #a1a100;">org.springframework.mail.javamail.JavaMailSender</span> <span style="color: #000000; font-weight: bold;">import</span> <span style="color: #a1a100;">org.springframework.mail.javamail.JavaMailSenderImpl</span> <span style="color: #000000; font-weight: bold;">import</span> <span style="color: #a1a100;">org.springframework.mail.javamail.MimeMessageHelper</span> &nbsp; <span style="color: #000000; font-weight: bold;">class</span> EmailSenderImpl <span style="color: #000000; font-weight: bold;">implements</span> EmailSender <span style="color: #66cc66;">&#123;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">private</span> <span style="color: #000000; font-weight: bold;">static</span> JavaMailSender sender &nbsp; 	EmailSenderImpl<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		sender <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> JavaMailSenderImpl<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span> 		sender.<span style="color: #006600;">host</span> <span style="color: #66cc66;">=</span> <span style="color: #ff0000;">&quot;smtp.myserver.fr&quot;</span> 	<span style="color: #66cc66;">&#125;</span> &nbsp; 	<span style="color: #000000; font-weight: bold;">def</span> send<span style="color: #66cc66;">&#40;</span>Email email<span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#123;</span> 		MimeMessageHelper message <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> MimeMessageHelper<span style="color: #66cc66;">&#40;</span>sender.<span style="color: #006600;">createMimeMessage</span><span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span>, <span style="color: #000000; font-weight: bold;">true</span><span style="color: #6
6cc66;">&#41;</span> 		message.<span style="color: #006600;">setText</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">body</span>, <span style="color: #000000; font-weight: bold;">false</span><span style="color: #66cc66;">&#41;</span> 		message.<span style="color: #006600;">setFrom</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">from</span><span style="color: #66cc66;">&#41;</span> 		message.<span style="color: #006600;">setSubject</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">subject</span><span style="color: #66cc66;">&#41;</span> 		message.<span style="color: #006600;">setTo</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">to</span> <span style="color: #000000; font-weight: bold;">as</span> <span style="color: #aaaadd; font-weight: bold;">String</span><span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span><span style="color: #66cc66;">&#41;</span> 		message.<span style="color: #006600;">setBcc</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">bcc</span> <span style="color: #000000; font-weight: bold;">as</span> <span style="color: #aaaadd; font-weight: bold;">String</span><span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span><span style="color: #66cc66;">&#41;</span> 		message.<span style="color: #006600;">setCc</span><span style="color: #66cc66;">&#40;</span>email.<span style="color: #006600;">cc</span> <span style="color: #000000; font-weight: bold;">as</span> <span style="color: #aaaadd; font-weight: bold;">String</span> <span style="color: #66cc66;">&#91;</span><span style="color: #66cc66;">&#93;</span><span style="color: #66cc66;">&#41;</span> 		email.<span style="color: #006600;">attachedFile</span>.<span style="color: #663399;">each</span> <span style="color: #66cc66;">&#123;</span> 			<span style="color: #aaaadd; font-weight: bold;">File</span> file <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> <span style="color: #aaaadd; font-weight: bold;">File</span><span style="color: #66cc66;">&#40;</span><span style="color: #000000; font-weight: bold;">new</span> URI<span style="color: #66cc66;">&#40;</span>it<span style="color: #66cc66;">&#41;</span><span style="color: #66cc66;">&#41;</span> 			message.<span style="color: #006600;">addAttachment</span><span style="color: #66cc66;">&#40;</span>file.<span style="color: #006600;">name</span>, file<span style="color: #66cc66;">&#41;</span> 		<span style="color: #66cc66;">&#125;</span> 		sender.<span style="color: #006600;">send</span><span style="color: #66cc66;">&#40;</span>message.<span style="color: #006600;">mimeMessage</span><span style="color: #66cc66;">&#41;</span> 	<span style="color: #66cc66;">&#125;</span> &nbsp; <span style="color: #66cc66;">&#125;</span></pre> <p>Jusqu'à présent nous n'avions d'adhérence que sur les librairies standard Groovy et Java. L'implémentation ci-dessus tire de nouvelles dépendances. Nous allons indiquer à groovy la liste des dépendences afin qu'il puisse les résoudre durant l'éxécution en annotant la classe <code>EmailSenderImpl</code>.</p> <pre class="groovy code groovy" style="font-family:inherit">@Grapes<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#91;</span> 	@Grab<span style="color: #66cc66;">&#40;</span>group<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'org.springframework'</span>, module<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'spring-beans'</span>, version<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'3.1.0.RELEASE'</span><span style="color: #66cc66;">&#41;</span>, 	@Grab<span style="color: #66cc66;">&#40;</span>group<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'org.springframework'</span>, module<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'spring-context-support'</span>, version<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'3.1.0.RELEASE'</span><span style="color: #66cc66;">&#41;</span>, 	@Grab<span style="color: #66cc66;">&#40;</span>group<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'javax.mail'</span>, module<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'mail'</span>, version<span style="color: #66cc66;">=</span><span style="color: #ff0000;">'1.4.4'</span><span style="color: #66cc66;">&#41;</span> <span style="color: #66cc66;">&#93;</span><span style="color: #66cc66;">&#41;</span></pre> <p>Pour finir, on remplace le stub par l'implémentation réelle.</p> <pre class="groovy code groovy" style="font-family:inherit">Email.<span style="color: #006600;">sender</span> <span style="color: #66cc66;">=</span> <span style="color: #000000; font-weight: bold;">new</span> EmailSenderImpl<span style="color: #66cc66;">&#40;</span><span style="color: #66cc66;">&#41;</span></pre> <h3>Pour conclure</h3> <p>J'ai souhaité aborder le sujet de manière pratique sans véritablement entrer dans la mécanique interne du langage qui n'est pas forcément évidente à appréhender en première approche. Nous avons pu voir que Groovy est adapté à l'implémentation de DSL de part sa syntaxe, son côté dynamique et ses closures.</p> <p>Un DSL étant par définition spécifique, nos éditeurs et autres ateliers de développement ne supportent pas nativement ces langages pour fournir des services de coloration syntaxique, d'auto-complétion ou encore de documentation en ligne. Pour résoudre ce problème, Groovy permet de définir des grammaires pour les DSL appelées DSLD (le dernier "D" pour "Description") qui sont exploitées nativement par le plugin Groovy-Eclipse, ce qui offre un bon point en plus pour Groovy en matière de DSL.</p> <p>Bref, cet article ne montre évidement qu'une petite partie de ce qu'il est possible de faire mais j'espère vous avoir convaincu de l'intérêt de Groovy pour développer des DSL.</p> <p>Voici le <a href="/wp-content/uploads/2015/07/EmailDsl.groovy">code source</a> complet de cet article.</p>