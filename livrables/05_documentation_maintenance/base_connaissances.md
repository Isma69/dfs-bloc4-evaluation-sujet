Base de connaissances — 

Note de passation

Ce document est destiné à un pair chargé de reprendre la maintenance de l'application. Il doit permettre de comprendre le fonctionnement, les points d’attention et les procédures essentielles sans zone d’ombre majeure.

1. Présentation de l’application

L’application est une API backend développée avec Laravel permettant la gestion de tickets et d’interventions.
Fonctionnalités principales :
gestion des tickets (création, consultation, mise à jour)
filtrage des tickets (recherche, priorité)
gestion des interventions liées aux tickets
intégration via webhook pour mise à jour externe
journalisation des événements (MongoDB)
L’application est exposée via HTTP/HTTPS et consommée via API REST.

2. Architecture technique

2.1 Composants principaux

Composant	Technologie	Rôle

Backend	Laravel (PHP 8)	logique métier et API REST
Serveur web	Apache 2.4	exposition HTTP/HTTPS
Base de données	MySQL	stockage des données métier
Logs	MongoDB	journalisation des événements
Déploiement	Git + SSH	mise à jour du code
Système	Ubuntu	hébergement

2.2 Schéma d’architecture

Client (API / navigateur)
        |
        v
   Apache (HTTP/HTTPS)
        |
        v
   Laravel (API)
        |
        +--> MySQL (tickets, interventions)
        |
        +--> MongoDB (logs)

3. Points d’attention connus

Architecture mono-serveur → point de défaillance unique
Webhook sensible (authentification + validation obligatoire)
Dépendance à l’ordre des migrations
Gestion des entrées API à surveiller (types stricts)
Logs MongoDB non critiques (fallback prévu)
Absence de pipeline CI/CD complet (déploiement manuel)

4. Procédures opérationnelles

4.1 Déploiement

cd /var/www/opstrack
git pull
composer install --no-dev --optimize-autoloader
php artisan optimize:clear
php artisan migrate --force
sudo systemctl reload apache2
Points de vigilance :
vérifier .env
vérifier les droits (storage, bootstrap/cache)
tester l’API après déploiement

4.2 Sauvegarde et restauration

Sauvegarde :
mysqldump -u user -p opstrack > backup.sql
Restauration :
mysql -u user -p opstrack < backup.sql
Code :
git clone <repo>

4.3 Supervision et alertes

Logs principaux :

Laravel :
/var/www/opstrack/storage/logs/laravel.log
Apache :
/var/log/apache2/opstrack-error.log

Commandes utiles :
tail -f laravel.log
tail -f opstrack-error.log
Tests rapides :
curl -I http://localhost
curl -H "Authorization: Bearer ..." http://localhost/api/v1/tickets

4.4 Accès et secrets

accès SSH par clé

.env contient :

DB credentials
API token
webhook credentials
ne jamais versionner .env
permissions restreintes

5. Bugs et failles corrigés pendant l’épreuve

Principaux problèmes rencontrés et corrigés :
erreur migration (clé étrangère → ordre incorrect des tables)
erreur 500 webhook (mauvaise initialisation Laravel)
erreur config() non disponible
erreur CSRF (419)
mauvaise gestion des inputs API (priority[])
erreur 302 liée au format de réponse
Toutes les corrections ont été validées par tests :
curl
base MySQL
Tinker

6. Améliorations recommandées

mise en place d’un pipeline CI/CD (GitHub Actions)
dockerisation de l’application
externalisation de la base de données
mise en place d’un monitoring (Grafana / Prometheus)
sauvegardes automatisées externalisées
mise en place d’un load balancer
gestion centralisée des logs

7. Contacts et ressources

Ressources utiles :
documentation Laravel : https://laravel.com/docs
documentation Apache : https://httpd.apache.org/docs/
documentation MySQL : https://dev.mysql.com/doc/

Accès projet :

dépôt Git (fork personnel du candidat)
serveur accessible via SSH
