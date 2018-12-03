# Cours 2018
## La sécurité sur le web
## Introduction

De la même manière qu’il propose un nombre presque infini de fonctionnalités, le web peut porter atteinte à notre sécurité, et les failles de sécurité qui lui sont liées sont très nombreuses. Pour s’en prémunir et naviguer en toute sécurité, il faut être au courant des risques et les comprendre. Ce cours a pour but d’avertir sur ces risques, d’expliquer les mécanismes du web qui présentent des failles et sont par conséquent un danger potentiel pour les internautes. Ce cours présente également une solution de sécurité : le VPN, qui permet entre autres de cacher ses informations.

## Les Menaces et failles visant la sécurité du Web

La conception et l’utilisation du Web exige la sécurité avec une vigilance car les la plupart des activités de ces jours se font sur l’internet et le Web. Dans ce cours, on se base sur les failles et les vulnérabilités sur le Web qui nous exigent la sécurité de nos sites Web et applications web.

## 1.  La faille Cross-Site Scripting(XSS)

### Definition

C’est une faille permettant l’injection du contenu du web via le code HTML ou Javascript dans des variables mal protégées. Il y a deux types de XSS:

### A-Le XSS réfléchi(non permanent)

Une faille XSS consiste à injecter du code frauduleux directement interprétable par le navigateur Web (par exemple du JavaScript ou du HTML). Cette attaque concerne plutôt la partie client c'est-à-dire vous (ou plutôt votre navigateur) puisqu'elle se base sur les informations que vous lui avez fournies au biais des champs à remplir de l’application web. Le navigateur ne fera aucune différence entre le code du site et celui injecté par le pirate, il va donc l'exécuter à votre insu. Les possibilités sont nombreuses : redirection vers un autre site, vol de cookies, modification du code HTML de la page, exécution d'exploits contre le navigateur : en bref, tout ce que ces langages de script vous permettent de faire.  

Source Image:Failles de sécurité des applications Web
 
### B-Le XSS stocké(permanent)

C’est lorsque le code malicieux est stocké sur les bases de donnée du serveur externe (l’application web). 
Il sera donc récupéré et exécuté à chaque fois par n’importe quel utilisateur lorsque ce dernier lance le site web ou l’application web. En effet, l’application va  accepter d’exécuter cet ordre comme si la demande provenait de l’utilisateur lors de son utilisation de l’application.

*Comment s’en protéger? :*

Plusieurs techniques permettent d'éviter le XSS :
Retraiter systématiquement le code HTML produit par l'application avant l'envoi au navigateur.
Filtrer les variables affichées ou enregistrées avec des caractères '<' et '>' . De façon plus générale, donner des noms préfixés aux variables contenant des chaînes venant de l'extérieur pour les distinguer des autres  (par exemple le préfixe &amp; au & (ET commercial) ), et ne jamais utiliser aucune des valeurs correspondantes dans une chaîne exécutable sans filtrage préalable. Certaines fonctions ont été conçues pour cette fin là comme  htmlspecialchars() qui convertit les caractères spéciaux en entités HTML,  la fonction htmlentities() qui est identique à htmlspecialchars() sauf qu’elle filtre tous les caractères équivalents au codage html ou javascript et strip_tags()  qui supprime toutes les balises.

## 2.  L’injection SQL

### Définition:

C’est un vecteur d’attaque basée sur une modification de requête sql qui permet aux utilisateurs malveillants d’injecter des morceaux de code non filtrés et de les exécuter. Une attaque par injection réussie peut usurper des identités, créer de nouvelles identités avec des droits d’administration, accéder à toutes les données sur le serveur ou détruire / modifier les données pour les rendre inutilisables.
Cette attaque se fait généralement par le biais d’un formulaire.  
  
Source image:Deep into SQL injection SQL Injection
 
Exemple basique de requête qui permet la connexion à un espace membre :

``` php
// On récupère les variables envoyées par le formulaire
$login = $_POST['login'];
$password = $_POST['password'];

// Connexion à la BDD en PDO
try { $bdd = new PDO('mysql:host=localhost;dbname=bdd','root',''); }
catch (Exeption $e) { die('Erreur : ' .$e->getMessage())  or die(print_r($bdd->errorInfo())); }

// Requête SQL
$req = $bdd->query("SELECT * FROM utilisateurs WHERE login='$login' AND password='$password'");

```
 

La requête va aller chercher dans la table "utilisateurs" une entrée où le pseudo est égal à $pseudo et où le mot de passe est égal à $password. La faiblesse de ce code se trouve dans le fait que l'on peut envoyer n'importe quoi par le biais du formulaire, y compris des morceaux de code. Par exemple, imaginez qu'un utilisateur (pour une raison x ou y) décide de mettre en login "jean' #" et laisser le password vide. Notre requête deviendrait donc:

``` php
<?php

$req = $bdd->query("SELECT * FROM utilisateurs WHERE login='jean' # AND password=''");

// Qui sera interprété de la façon suivante

$req = $bdd->query("SELECT * FROM utilisateurs WHERE login='jean'");

?>
```

 le symbole # permet de faire un commentaire en PHP. Tout ce qui suit ce symbole est donc considéré par PHP comme un commentaire et n'est pas pris en compte dans la requête SQL. Grâce à cette injection l'utilisateur va pouvoir se connecter à n'importe quel compte sans connaître son mot de passe.    
 
*Comment s’en protéger ? :*

Pour s’en protéger, il suffit d’utiliser des requêtes préparées :

``` php
<?php

// On récupère les variables envoyées par le formulaire
$login = $_POST['login'];
$password = $_POST['password'];

// Connexion à la BDD en PDO
try { $bdd = new PDO('mysql:host=localhost;dbname=bdd','root',''); }
catch (Exeption $e) { die('Erreur : ' .$e->getMessage())  or die(print_r($bdd->errorInfo())); }

// Requête SQL sécurisée
$req = $bdd->prepare("SELECT * FROM utilisateurs WHERE login= ? AND password= ?");
$req->execute(array($login, $password));

?>
```

# 3. La faille Include
## 3.1 Définition :
La faille include consiste à l’usage frauduleux de la fonction Include(). C’est une fonction utilisée dans pas mal de langage de programmation pour exécuter du code PHP situé dans une autre page, autrement dit, inclure une page dans une autre. Comme cette fonction permet l'accès à d’autres pages, elle offre aux hackers la possibilité d’accéder aux données sur le serveur. Il ya deux types de faille Include:
*	La faille include à distance: C’est à la fois la plus fréquente et la plus facile à exploiter. Le Hacker peut inclure n’importe quelle page au biais de la fonction, y compris une page “backdoor” sur le serveur. Celle-ci va donner des droits privilégiés à son auteur et va donc lui permettre d’accéder aux données du site et de le modifier.
*	La faille include en local: À part inclure une backdoor, la fonction include() permet de naviguer sur les répertoires du site et récupérer facilement le fichier des passwords ..

## 3.2 Comment s’en protéger ? 
Il faut d’abord détecter la faille sur le site. Pour ce faire, on essaye d’inclure une page inexistante et le message d’erreur php sera la preuve que la faille existe. Ensuite, il faudra rédiger un fichier htaccess de configuration qui permettra de gérer les droits d’accès aux sites et grâce au code ci-dessous, on filtrera les pages incluses:

# 4. L’Attaque par force brute
## 4.1 Définition :
L’attaque par force brute est utilisé pour trouver un mot de passe ou une clé en testant une par une et tous les combinaisons possibles. Il nécessite effectivement beaucoup de temps d'exécution mais il est efficace. Cette attaque aussi est souvent combiné avec l’attaque par dictionnaire dont l’attaquant utilise d’énormes dictionnaires contenant des mots de passes régulièrement utilisé.

## 4.2 Comment s’enprotéger ?

Se défendre contre l’attaque par force brute :

### La vérification par captcha :

Ce sont de petites cases avec des lettres déformées que vousdevez recopier pour confirmer que vousêtes un humain.
![image](https://assets.change.org/photos/5/lh/ou/BElHoUUqfADSIGU-800x450-noPad.jpg?1519282724)  
[Voici le lien de cette image](https://assets.change.org/photos/5/lh/ou/BElHoUUqfADSIGU-800x450-noPad.jpg?1519282724) !
Cette dernière technique est très simple. Elle consiste à insérer un captcha de vérification dans vos formulaires. C'est presque imparable pour être certain qu'on a à faire à un humain et non à un script.
### Utilisation de mots de passes robustes pour une durée limitée :
Elle consiste à renforcer le mot de passe en évitant les écueils qui exploitent les attaques par force brute optimisée. Renforcer la force brute du mot de passe consisteà :
* allonger le mot de pass eou la clé si cela est possible
* utiliser la plus grande gamme de symboles possibles (minuscules, majuscules, ponctuations, chiffres)  

De plus, l’utilisateur sera forcé à changer son mot de passe après une certaine période définie par le développeur. Ce nouveau mot de passe ne pourra pas être un mot de passe qui a été déjà utilisé par cetutilisateur.

### Authentication multiple :
L’authentification multiple (Multi-Factor Authentication enanglais qui correspond à MFA) est un système de sécurité qui fait appel à plusieurs méthodes d’authentification, à partir de différentes catégories d’informations d’identification (des preuves), pour vérifier l’identité de l’utilisateur qui souhaite se connecter.  


# 5. Les Virus
## 5.1 Définition :

De nos jours, les pirates informatiques, les auteurs de virus, les spammers et les développeurs de programmes espions sont regroupés sous le nom de « cybercriminels ». Les menaces Web aident ces individus à atteindre un objectif précis. L'un de ces objectifs est de voler des informations à des fins de revente.
Il existe des dizaines de milliers de virus/programmes malveillants et de nouveaux sont créés chaque jour, les virus informatiques de notre époque peuvent provoquer des dommages importants en exploitant les failles de sécurité des réseaux d'entreprise, des systèmes de messagerie électronique et des sites Web. En effet, c’est un petit programme qui s’attache à un programme complet et le modifie et donc mène à une perturbation. En exécutant le programme, il y a exécution du virus en parallèle afin de contaminer le système. De plus, tous les moyens d’échange de données numériques comme les réseaux informatiques, le cédérom, les clefs USB…favorisent la diffusion du programme malveillants. Il est susceptible d'altérer le fonctionnement de votre ordinateur, de détruire des informations, voire d'en récupérer pour les transmettre à distance. Souvent, l’utilisateur déclenche lui-même l’activation du virus en ouvrant un fichier contaminé.  
 
 Voici quelques exemples de virus connus:
 
* Cheval de Troie : Un type de logiciel malveillant qui utilise un code malveillant pour installer des logiciels qui semble être bien, mais est masqué pour créer portes arrière dans un système,afin d’entraîner la perte ou le vol des données à partir d'une source externe.
* Ransomware : Il infecte l’ordinateur, puis chiffre les données telles que les documents personnels, photos… et puis demandent un racon pour les récupérer. Si vous refusez d payer, les données seront supprimées.
* Adware : Les programmes envoient automatiquement des publicités aux ordinateurs. Bien que certains adware soient relativement sans danger, d'autres variantes utilisent des outils de suivi permettant de récupérer des informations sur votre site ou sur votre historique de navigation.  

## 5.2 Comment s’en protéger ?
Les applications web, faisant partie intégrante des processus métiers, sont une cible facile pour les cybercriminels lorsqu’elles ne sont pas sécurisées. Une fois que les menaces sont détectées, elles peuvent se propager sur tout le système et son élimination complète demande un temps et des efforts considérables. En outre, les systèmes n’appliquant pas des stratégies de sécurité bien strictes, sont les plus exposées aux menaces.  
De plus, le meilleur moyen de se protéger contre les virus est d’équiper votre ordinateur d’un anti-virus. Ce dernier, a pour mission d’analyser tous les fichiers entrant sur votre ordinateur et d’identifier ceux qui sont infectés. En adoptant les bons gestes, et donc en respectant la mise à jour du antivirus, le fait de ne pas ouvrir les mails ou les fichiers ou même les sites internet suspicieux contribuent à la protection des menaces. De plus, un pare-feu peut aussi jouer un rôle dans la protection, car il peutsignaler une activité suspecte lorsqu’un virus ou un ver tente de se connecter à votre ordinateur. Un pare-feu peut également empêcher les virus, les vers et les pirates informatiques de télécharger sur votre ordinateur des programmes potentiellement malveillants.  

![image](https://upload.wikimedia.org/wikipedia/commons/5/53/Conficker.svg)
[Voici le lien de cette image](https://upload.wikimedia.org/wikipedia/commons/5/53/Conficker.svg) !
