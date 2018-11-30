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
