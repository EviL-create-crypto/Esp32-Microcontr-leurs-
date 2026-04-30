Ce projet consiste à réaliser un système IoT avec une carte ESP32 capable de communiquer avec un serveur web pour envoyer, stocker et afficher des données en temps réel. L’ESP32 se connecte au WiFi, génère des valeurs et les transmet au serveur via des requêtes HTTP. Côté serveur, développé en PHP, les données sont enregistrées dans des fichiers afin de conserver la dernière valeur ainsi qu’un historique horodaté. Une interface web permet d’afficher ces informations dynamiquement grâce à AJAX, sans rechargement de la page. Le système permet également de contrôler l’ESP32 à distance, notamment pour piloter une LED ou un buzzer via des boutons sur la page web. La réalisation a nécessité la configuration du serveur Apache, la gestion des droits d’écriture et la mise en place de la communication entre les différents éléments. Ce projet m’a permis de comprendre le fonctionnement global d’un système connecté, incluant l’envoi de données, leur traitement côté serveur et le contrôle d’actionneurs à distance, constituant ainsi une base solide pour des applications IoT plus avancées.

Dans ce TP, nous apprenons à faire communiquer un ESP32 avec un serveur web en utilisant le WiFi afin d’envoyer, stocker et afficher des données.

Tout d’abord, nous connectons l’ESP32 à l’ordinateur et nous le programmons avec Arduino IDE en modifiant les paramètres suivants : **(Nom du réseau, mot de passe ainsi que l’adresse du serveur : IP de la VM / dossier / page PHP)**. Nous choisissons la *carte ESP32 Dev Module* et *le port dédié*, puis nous configurons la connexion WiFi en indiquant le **nom du réseau** et **le mot de passe**. Une fois connecté, l’ESP32 affiche son *adresse IP* dans le moniteur série, ce qui permet de l’identifier sur le réseau.

Ensuite, sur la machine virtuelle Ubuntu, nous créons un dossier dans un *dossier commun* contenant les fichiers suivants : **(data.php, valeur.csv, valeur.txt et index.php)**. Le fichier *data.php* sert à recevoir les données envoyées par l’ESP32. Nous testons son fonctionnement en ouvrant son *adresse* dans un navigateur web **(IP de la VM / dossier / page PHP)**.

Après cela, nous programmons l’ESP32 pour envoyer des données au serveur. Lorsqu’il est connecté au WiFi, il génère une valeur aléatoire entre 0 et 100, puis l’envoie au serveur Ubuntu via une requête HTTP de type POST. Une pause de quelques secondes est ajoutée entre chaque envoi pour éviter de surcharger le serveur.

Sur le serveur, le fichier *data.php* récupère la valeur envoyée et l’enregistre dans le fichier **valeur.txt**. Une version améliorée permet également d’enregistrer chaque valeur avec la date et l’heure dans le fichier *valeur.csv*, afin de conserver un historique.

Enfin, nous affichons ces données sur une page web avec **index.php**. Ce fichier lit la dernière valeur enregistrée dans **valeur.txt** et l’affiche sur la page. La page se met à jour automatiquement toutes les deux secondes pour afficher les nouvelles valeurs.

Ce TP permet de comprendre le fonctionnement d’un objet connecté : l’envoi de données par WiFi, leur stockage sur un serveur, puis leur affichage en temps réel sur une page web.

Ce TP montre le fonctionnement de base d’un objet connecté :
**capteur** → **WiFi** → **serveur** → **base de données** → **page web**




