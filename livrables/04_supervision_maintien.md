Supervision, journalisation, sauvegarde et maintenance corrective

Compétence évaluée : C32 — Mettre en œuvre un système de supervision pour détecter, diagnostiquer et corriger bugs, incidents et failles.

1. Journalisation

1.1 Services journalisés

Service	Emplacement des journaux	Niveau de détail
Laravel	/var/www/opstrack/storage/logs/laravel.log	erreurs applicatives, exceptions, stack trace
Apache (vhost)	/var/log/apache2/opstrack-error.log	erreurs PHP, erreurs serveur
Apache global	/var/log/apache2/error.log	erreurs système et HTTP
MySQL	logs MySQL	erreurs base de données
MongoDB	base MongoDB (EventLog)	logs métier applicatifs

1.2 Configuration de la journalisation

Laravel configuré en mode production
Logs écrits automatiquement via Monolog
Service dédié EventLogService pour journaliser :
événements API
événements webhook
Fallback prévu :
si MongoDB indisponible → log Laravel (warning)

2. Outils et configurations d’audit

Outils utilisés :
tail -f pour suivi temps réel des logs
grep pour filtrer des erreurs spécifiques
curl pour tester les endpoints
mysql CLI pour vérifier les données
php artisan tinker pour tester directement le modèle
Ces outils ont permis :
d’identifier rapidement les erreurs 500 ;
de tracer les causes (migration, config, CSRF, etc.) ;
de valider les corrections.

3. Supervision et alertes

3.1 Sondes mises en place

Sonde	Cible	Seuil ou condition	Action en cas d’alerte
HTTP	/	code ≠ 200	vérifier Apache / logs
API	/api/v1/tickets	code ≠ 200/401	vérifier token / backend
Webhook	/hooks.php	code ≠ 200	vérifier auth / CSRF
Base de données	MySQL	erreur connexion	vérifier service MySQL
Logs Laravel	erreurs critiques	présence d’exception	analyse immédiate

3.2 Mécanisme d’alerte

Dans ce contexte :
supervision manuelle
alertes via observation logs et retours HTTP
Améliorations possibles :
intégration Prometheus / Grafana
alertes email / Slack
monitoring uptime automatisé

4. Stratégie de sauvegarde et restauration

4.1 Éléments sauvegardés

Élément	Méthode	Fréquence	Rétention
Base MySQL	dump SQL (mysqldump)	quotidien	7 jours
Code source	Git	continu	illimité
Fichier .env	sauvegarde manuelle sécurisée	ponctuel	sécurisé
Logs importants	archivage	hebdomadaire	1 mois

4.2 Procédure de restauration

Restaurer la base :
mysql -u user -p opstrack < backup.sql
Restaurer le code depuis Git :
git pull
Vérifier .env
Relancer :
php artisan migrate
Redémarrer Apache

5. Diagnostic et correction du bug technique

5.1 Symptôme observé

erreur HTTP 500 sur certaines requêtes
erreur lors des migrations :
Failed to open the referenced table 'tickets'

5.2 Démarche de diagnostic

consultation des logs Laravel :
tail -n 50 storage/logs/laravel.log
analyse de la stack trace
identification d’un problème de clé étrangère
vérification des migrations

5.3 Cause racine identifiée

La table interventions était créée avant la table tickets, alors qu’elle contient une clé étrangère :
$table->foreignId('ticket_id')->constrained()
→ MySQL ne trouvait pas la table référencée.

5.4 Correctif appliqué

correction de l’ordre des migrations
suppression / recréation correcte de la base
relance :
php artisan migrate

5.5 Vérification après correction

migration exécutée sans erreur
création d’intervention test via Tinker :
Intervention::create([...])
fonctionnement validé

6. Diagnostic et correction de la faille de sécurité

6.1 Faille identifiée
Le webhook présentait plusieurs problèmes :
accès direct via hooks.php sans bootstrap Laravel complet
erreur critique :
Class "config" does not exist
absence de gestion CSRF correcte
endpoint exposé sans protection adaptée au début

6.2 Démarche de diagnostic

analyse logs Apache :
/var/log/apache2/opstrack-error.log
identification de l’erreur sur config()
test via curl avec authentification
reproduction du bug

6.3 Évaluation du risque

risque d’exécution incorrecte du webhook
risque de contournement de sécurité
indisponibilité fonctionnelle du service

6.4 Mesure corrective appliquée

Corrections mises en place :
utilisation correcte du framework Laravel (route contrôlée)
mise en place de Basic Auth sécurisée
exemption CSRF pour le webhook
validation stricte des entrées :
$request->validate([...])
mise à jour du ticket + création intervention sécurisée

6.5 Vérification après correction

Test :
curl -u secure_user:password ...
Résultat :
réponse JSON valide
intervention créée
ticket mis à jour en base :
SELECT status FROM tickets WHERE reference='INC-240302';

7. Autres observations

erreur 302 détectée sur API → liée à redirection HTML vs JSON
correction via header :
Accept: application/json
validation des entrées incorrectes :
priority[]=high → retourne 422 (comportement attendu)
amélioration du contrôleur tickets :
sécurisation des filtres
mise en place de logs métiers via MongoDB
nettoyage du cache après déploiement :
php artisan optimize:clear
