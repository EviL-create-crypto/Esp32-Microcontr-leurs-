# Esp32-Microcontr-leurs-
Ce projet a pour objectif de réaliser un système IoT complet basé sur une carte ESP32, capable de communiquer avec un serveur web afin d’envoyer, stocker et afficher des données en temps réel. L’ESP32 joue le rôle de microcontrôleur principal : il se connecte à un réseau WiFi, génère des valeurs aléatoires et les transmet à un serveur via des requêtes HTTP. Ces données sont ensuite enregistrées côté serveur dans des fichiers texte et CSV afin de conserver à la fois la dernière valeur et un historique des mesures.

Le serveur web, développé en PHP, reçoit les données envoyées par l’ESP32 et les traite automatiquement. Il stocke la dernière valeur dans un fichier texte et ajoute chaque nouvelle donnée dans un fichier CSV horodaté. Une interface web permet ensuite d’afficher ces informations en temps réel grâce à AJAX, sans avoir besoin de recharger la page. Cette mise à jour dynamique permet de visualiser l’évolution des données de manière fluide et continue.

Le projet intègre également un système de commande à distance de l’ESP32. Depuis la page web, l’utilisateur peut envoyer des ordres pour contrôler des actionneurs comme une LED ou un buzzer. Ces commandes sont envoyées via des requêtes HTTP vers l’ESP32, qui agit en conséquence en activant ou désactivant les composants. Cela permet de créer une interaction directe entre l’utilisateur et le microcontrôleur.

Durant la réalisation de ce projet, plusieurs difficultés ont été rencontrées, notamment la configuration du serveur Apache pour exécuter correctement le PHP, la gestion des permissions d’écriture sur les fichiers, ainsi que la communication entre l’ESP32 et l’interface web. Des problèmes liés à l’utilisation des fonctions ESP32 (compatibilité des bibliothèques PWM) ont également nécessité des ajustements.

Ce projet m’a permis de comprendre le fonctionnement global d’un système IoT, depuis la collecte de données jusqu’à leur traitement et leur affichage. J’ai appris à manipuler les communications HTTP, à stocker des données côté serveur, à utiliser AJAX pour le temps réel, et à contrôler des actionneurs à distance. Il m’a également permis de développer ma capacité à analyser et résoudre des problèmes techniques sur un système complet.

Enfin, ce travail m’a apporté une meilleure compréhension de l’architecture des systèmes connectés et des interactions entre un microcontrôleur, un serveur web et une interface utilisateur. Il constitue une base solide pour la réalisation de projets IoT plus avancés.


#include <WiFi.h>
#include <HTTPClient.h>
#include <WebServer.h>

// ======================================================
//  CONFIGURATION WIFI
// ======================================================
// Nom du réseau WiFi (SSID)
const char* ssid = "TPSN035";

// Mot de passe WiFi
const char* password = "BTSSN2022";


// ======================================================
//  SERVEUR WEB PHP (RECEPTION DES DONNÉES)
// ======================================================
// URL du serveur où l’ESP32 envoie les valeurs aléatoires
const char* serverName = "http://192.168.100.35/btsciel/data.php";


// ======================================================
//  SERVEUR WEB INTERNE ESP32
// ======================================================
// Permet de créer des routes comme /led et /son
WebServer server(80);


// ======================================================
//  PINS UTILISÉES
// ======================================================
// LED branchée sur GPIO 27
const int led = 27;

// Buzzer branché sur GPIO 26
const int buzzer = 26;


// ======================================================
//  TIMER POUR ENVOI AUTOMATIQUE
// ======================================================
// Stocke le dernier temps d’envoi
unsigned long previousMillis = 0;

// Intervalle d’envoi (5 secondes)
const long interval = 5000;


// ======================================================
//  CONNEXION WIFI
// ======================================================
// Fonction qui connecte l’ESP32 au réseau WiFi
void connectWiFi() {

  // Lancement de la connexion
  WiFi.begin(ssid, password);

  Serial.print("Connexion WiFi");

  // Attente de connexion
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  // Connexion réussie
  Serial.println("\nConnecté au WiFi !");
  Serial.print("Adresse IP ESP32 : ");
  Serial.println(WiFi.localIP());
}


// ======================================================
//  ENVOI DE DONNÉES VERS LE SERVEUR PHP
// ======================================================
// Génère une valeur aléatoire et l’envoie au serveur
void sendData() {

  // Génération d’une valeur entre 0 et 100
  int valeur = random(0, 100);

  // Création client HTTP
  HTTPClient http;

  // Connexion au serveur PHP
  http.begin(serverName);

  // Type de contenu envoyé
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");

  // Donnée envoyée (format POST)
  String postData = "valeur=" + String(valeur);

  // Envoi de la requête POST
  int code = http.POST(postData);

  // ======================================================
  // AFFICHAGE DEBUG SERIAL
  // ======================================================
  Serial.println("----------------------");
  Serial.print("Valeur envoyée : ");
  Serial.println(valeur);

  Serial.print("Code HTTP : ");
  Serial.println(code);

  // Si réponse serveur OK
  if (code > 0) {
    Serial.println("Réponse serveur : ");
    Serial.println(http.getString());
  }

  // Fermeture connexion HTTP
  http.end();
}


// ======================================================
//  ACTION LED
// ======================================================
// Fonction appelée quand on tape /led dans le navigateur
void handleLed() {

  // Allume la LED
  digitalWrite(led, HIGH);
  delay(300);

  // Éteint la LED
  digitalWrite(led, LOW);

  // Réponse envoyée au navigateur
  server.send(200, "text/plain", "LED OK");
}


// ======================================================
//  ACTION BUZZER
// ======================================================
// Fonction appelée quand on tape /son dans le navigateur
void handleBuzzer() {

  // Initialisation du PWM pour le buzzer
  // (compatible ESP32 core 3.x)
  ledcAttach(buzzer, 2000, 8);

  // Séquence sonore
  ledcWriteTone(buzzer, 1000);
  delay(300);

  ledcWriteTone(buzzer, 500);
  delay(300);

  ledcWriteTone(buzzer, 1500);
  delay(300);

  // arrêt du son
  ledcWriteTone(buzzer, 0);

  // Réponse au navigateur
  server.send(200, "text/plain", "BUZZER OK");
}


// ======================================================
// INITIALISATION (SETUP)
// ======================================================
void setup() {

  // Démarrage moniteur série
  Serial.begin(115200);

  // Configuration LED en sortie
  pinMode(led, OUTPUT);
  digitalWrite(led, LOW);

  // Connexion WiFi
  connectWiFi();

  // ======================================================
  // ROUTES WEB (API ESP32)
  // ======================================================
  // http://IP_ESP32/led  → LED
  server.on("/led", handleLed);

  // http://IP_ESP32/son  → BUZZER
  server.on("/son", handleBuzzer);

  // Démarrage serveur
  server.begin();

  Serial.println("Serveur ESP32 prêt !");
}


// ======================================================
//  BOUCLE PRINCIPALE
// ======================================================
void loop() {

  // Gestion des requêtes web entrantes
  server.handleClient();

  // Reconnexion automatique si WiFi perdu
  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
    return;
  }

  // ======================================================
  // ENVOI AUTOMATIQUE TOUTES LES 5 SECONDES
  // ======================================================
  if (millis() - previousMillis >= interval) {
    previousMillis = millis();

    // Envoi d’une nouvelle valeur au serveur
    sendData();
  }
}
