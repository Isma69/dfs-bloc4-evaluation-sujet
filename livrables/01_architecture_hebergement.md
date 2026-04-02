Architecture cible et choix de l’hébergement

Compétence évaluée : C29 — Sélectionner une plateforme d’hébergement adaptée aux exigences techniques, économiques, qualitatives et réglementaires

1. Analyse des besoins techniques

L’application à déployer est une API backend développée avec Laravel, exposant des endpoints REST pour la gestion de tickets et d’interventions.
Les besoins identifiés sont les suivants :
Hébergement d’une application PHP (Laravel) avec serveur web (Apache ou Nginx)
Base de données relationnelle (MySQL)
Accès sécurisé via HTTPS
Gestion d’authentification (API token + webhook sécurisé)
Capacité à recevoir des requêtes externes (webhooks)
Persistance des données
Journalisation des événements (logs applicatifs + MongoDB)
Disponibilité continue (service accessible en permanence)
Possibilité de montée en charge (scalabilité)
Contraintes :
Budget limité (solution simple mais fiable)
Mise en production rapide
Facilité d’administration (niveau junior)

2. Architecture cible proposée
L’architecture repose sur une machine virtuelle unique (VPS) hébergeant l’ensemble des composants nécessaires à l’application.

2.1 Diagramme de déploiement
[ Client / API Consumer ]
            |
            v
     HTTPS (443)
            |
            v
   [ Serveur VPS ]
   -------------------------
   | Apache / PHP         |
   | Laravel API          |
   |                      |
   | MySQL (DB)           |
   | MongoDB (logs)       |
   -------------------------
            |
            v
     Stockage local
     
2.2 Description des composants

Composant	Service ou technologie	Dimensionnement	Justification
Serveur	VPS Linux (Ubuntu)	2 vCPU / 2 Go RAM / 40 Go SSD	Suffisant pour une application API avec faible à moyenne charge
Serveur web	Apache 2.4	Inclus dans VPS	Compatible avec PHP et simple à configurer
Backend	PHP 8 + Laravel	Inclus	Framework utilisé par l’application
Base de données	MySQL	Inclus	Base relationnelle adaptée aux données métiers
Logs	MongoDB	Inclus	Permet de stocker les logs applicatifs
Sécurité	HTTPS (SSL)	Certificat Let’s Encrypt	Chiffrement des échanges
Accès	SSH sécurisé	Clé SSH	Administration serveur sécurisée

3. Choix du fournisseur et des services

3.1 Fournisseur retenu

Le fournisseur choisi est un VPS type OVHcloud / Scaleway / AWS Lightsail (équivalent).

3.2 Justification du choix

Bon rapport qualité/prix
Déploiement rapide d’une machine
Accès root complet (flexibilité maximale)
Compatible avec stack PHP / MySQL / Apache
Possibilité d’évolution (upgrade VPS)
Localisation en Europe (RGPD)
Le choix d’un VPS plutôt qu’un PaaS permet de garder un contrôle total sur l’environnement (utile dans un contexte d’apprentissage et de certification).

4. Estimation des coûts

Poste de dépense	Coût mensuel estimé	Coût annuel estimé
VPS (2 vCPU / 2 Go RAM)	~10 €	~120 €
Nom de domaine	~1 €	~12 €
Certificat SSL (Let’s Encrypt)	0 €	0 €
Stockage supplémentaire	inclus	inclus
Total	~11 €	~132 €

5. Élasticité et évolutivité

L’architecture peut évoluer facilement :
Upgrade vertical du VPS (RAM / CPU)
Séparation des services :
DB sur un serveur dédié
Backend sur plusieurs instances
Ajout d’un load balancer
Conteneurisation (Docker) possible à terme
Cette approche permet une montée en charge progressive sans refonte complète.

6. Disponibilité et continuité de service

Hébergement sur un datacenter fiable (SLA fournisseur)
Redémarrage automatique du serveur en cas de crash
Surveillance des services (Apache, MySQL)
Possibilité de mettre en place :
backups réguliers
duplication future (HA)

7. Sécurité et sauvegarde

Mesures mises en place :
Accès SSH sécurisé (clé, désactivation du mot de passe)
Firewall (UFW) :
22 (SSH)
80 (HTTP)
443 (HTTPS)
HTTPS obligatoire
Protection API via token
Authentification webhook via Basic Auth
Sauvegardes :
Dump régulier de la base MySQL
Sauvegarde des fichiers applicatifs
Possibilité d’externaliser (S3 / stockage distant)

8. Conformité et contraintes réglementaires

Hébergement en Europe (respect RGPD)
Données personnelles limitées et maîtrisées
Accès sécurisé aux données
Traçabilité via logs
Pas d’exposition de secrets dans le code
