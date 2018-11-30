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

Une faille XSS consiste à injecter du code frauduleux directement interprétable par le navigateur Web (par exemple du JavaScript ou du HTML). Cette attaque concerne plutôt la partie client c'est-à-dire vous (ou plutôt votre navigateur) puisqu'elle se base sur les informations que vous lui avez fournies au biais des champs à remplir de l’application web. Le navigateur ne fera aucune différence entre le code du site et celui injecté par le pirate, il va donc l'exécuter à votre insu. Les possibilités sont nombreuses : redirection vers un autre site, vol de cookies, modification du code HTML de la page, exécution d'exploits contre le navigateur : en bref, tout ce que ces langages de script vous permettent de faire.  Source Image:Failles de sécurité des applications Web
 
Le XSS stocké(permanent)
C’est lorsque le code malicieux est stocké sur les bases de donnée du serveur externe (l’application web). 
Il sera donc récupéré et exécuté à chaque fois par n’importe quel utilisateur lorsque ce dernier lance le site web ou l’application web. En effet, l’application va  accepter d’exécuter cet ordre comme si la demande provenait de l’utilisateur lors de son utilisation de l’application.
Comment s’en protéger? : 
Plusieurs techniques permettent d'éviter le XSS :
Retraiter systématiquement le code HTML produit par l'application avant l'envoi au navigateur.
Filtrer les variables affichées ou enregistrées avec des caractères '<' et '>' . De façon plus générale, donner des noms préfixés aux variables contenant des chaînes venant de l'extérieur pour les distinguer des autres  (par exemple le préfixe &amp; au & (ET commercial) ), et ne jamais utiliser aucune des valeurs correspondantes dans une chaîne exécutable sans filtrage préalable. Certaines fonctions ont été conçues pour cette fin là comme  htmlspecialchars() qui convertit les caractères spéciaux en entités HTML,  la fonction htmlentities() qui est identique à htmlspecialchars() sauf qu’elle filtre tous les caractères équivalents au codage html ou javascript et strip_tags()  qui supprime toutes les balises.








### 3.  Cross-Site Request Forgery(CSRF) ou One click attack
La faille CSRF peut être aussi expliquée comme la falsification inter-sites car il permet l’attaquant d'exécuter les actions en utilisant les informations de l’autre utilisateur sans son consentement c’est-à-dire que l’attaquant vise un site ou une page en utilisant l’utilisateur comme déclencheur sans qu’il le sache.
![image](https://images.leblogduhacker.fr/2014/02/faille-csrf1.jpg)

[Source image:La faille CSRF, explication et contre-mesures](https://www.leblogduhacker.fr/faille-csrf-explications-contre-mesures/) Suivant l’exemple ci-dessus l’attaquant peut supprimer l’article d’un administrateur. Si on suppose que le lien de suppression d’un article soit:http://www.monblog.fr/del.php?id=1, Normalement un visiteur non connecté à la page d’administration n’a pas le droit d’éditer ou de supprimer les articles d’un blog qui ne lui appartient pas. Par contre si le visiteur connaît ce lien ça suffit pour lui d’envoyer le lien à l’administrateur en faisant en sorte que ce dernier clique et quand l’administrateur clique le lien de suppression s'exécute avec succès.
#### Comment se protéger contre la faille CSRF ?
Demander des confirmations à l'utilisateur pour les actions critiques, au risque d'alourdir l'enchaînement des formulaires.
Utiliser des jetons de validité dans les formulaires : faire en sorte qu'un formulaire posté ne soit accepté que s'il a été produit quelques minutes auparavant : le jeton de validité en sera la preuve. Le jeton de validité doit être transmis en paramètre et vérifié côté serveur.
#### Un exemple pour l’authentification par jeton
Un jeton (aussi appelé **token** en anglais) est un nombre ou une chaîne de caractère aléatoire qui va être testée avant toute modification ou édition d’un article.
Il se présente habituellement sous forme de **hash md5** comme celui-ci :
``` php
b6cf20590a57f4685c9bdc6c53d12ff8
```
Ce token doit être crée dans un fichier PHP qui sera appelé sur toutes les pages. Typiquement, il s’agit d’un fichier du type config.php ou functions.php.
On génère souvent un nombre “aléatoire” avec des fonctions utilisant le temps en PHP. Par exemple on peut générer un jeton avec :
``` php
md5(uniqid(mt_rand(), true));
```
La fonction [uniqid()](http://fr2.php.net/manual/en/function.uniqid.php) génère un identifiant unique basé sur le temps en microsecondes. Cependant PHP ne recommande pas cette fonction car elle ne génère pas des chaînes impossible à deviner à l’avance.
Du coup, on va plutôt utiliser :
``` php
md5(bin2hex(openssl_random_pseudo_bytes(6)));
```
qui est cette fois hautement **sécurisé**.
La fonction [openssl_random_pseudo_bytes()](http://www.php.net/manual/fr/function.openssl-random-pseudo-bytes.php) génère une chaîne pseudo-aléatoire d’octets de taille 6 bits * 2 qu’on convertit ensuite en hexadécimal, 6 étant le nombre donné en paramètre de la fonction (on peut le changer).
Ainsi, dans un fichier PHP global on va écrire le code suivant :
``` php
<?php
if (!isset($_SESSION['jeton'])) {
   $_SESSION['jeton'] = bin2hex(openssl_random_pseudo_bytes(6));
}
?>
```
Ce code signifie : Si le jeton de session n’est pas encore défini, on le génère aléatoirement et on l’enregistre pour la session courante.
Ensuite, à chaque connexion d’un utilisateur, on va devoir générer un jeton **qui lui est propre**.
Pour cela, on peut avant chaque connexion régénérer le jeton, en **supprimant** le jeton de la session précédente :
``` php
unset($_SESSION['jeton']);
```
Il reste ensuite à modifier dynamiquement les liens de suppression, admettons que le lien précédent était écrit de la forme :
``` php
<a href="http://www.monblog.fr/del.php?id=<? echo get_article_id(); 
?>>Supprimer l'article</a>
```
On le remplace par :
``` php
<a href="http://www.monblog.fr/del.php?id=<? echo get_article_id() . 
"&jeton=". $_SESSION['jeton']; ?>>Supprimer l'article</a>
```
L’url de suppression s’afficher donc comme ceci :
``` php
http://www.monblog.fr/del.php?id=1&jeton=b6cf20590a57f4685c9bdc6c53d12ff8
```
Au lieu de s’afficher comme cela :
``` php
http://www.monblog.fr/del.php?id=1
```
Et enfin, dans notre fichier de suppression del.php, on va s’assurer qu’il existe un jeton et que ce jeton soit valide. Le fichier, avant toute modification se présentait comme cela :
``` php
<?php
if(isset($_GET['id'])) {
   supprimer_article($_GET['id']);
} else {
   die("Aucun ID d'article sélectionné.");
}
?>
```
Il devient donc :
``` php
<?php
if(isset($_GET['id']) && isset($_GET['jeton']) && 
($_GET['jeton'] == $_SESSION['jeton'])) {
   supprimer_article($_GET['id']);
} else {
   die("ID ou jeton de session invalide.");
}
?>
```
Ce qui  signifie : Si l’id de l’article est défini mais aussi le jeton et que ce jeton correspond au jeton de la session actuelle, alors on peut supprimer.
Dernière note, on utilise $_GET qui récupère les paramètres depuis une URL, il aurait été préférable est **encore plus sécurisé** d’utiliser $_POST avec un formulaire pour ne pas afficher de jeton dans les URLs.
