Déploiement automatisé

Compétence évaluée : C31 — Mettre en œuvre un système de déploiement automatisé respectant les bonnes pratiques DevOps.

1. Stratégie de déploiement

1.1 Vue d’ensemble
La stratégie retenue repose sur un déploiement simple, reproductible et adapté au contexte de l’épreuve.
Le principe est le suivant :
le code est versionné dans Git ;
le déploiement est déclenché manuellement ;
le serveur récupère la nouvelle version du code ;
les dépendances sont installées ;
les caches Laravel sont nettoyés ;
les migrations sont exécutées ;
des tests de bon fonctionnement sont réalisés après déploiement.
Dans le cadre de l’épreuve, je n’ai pas mis en place une chaîne CI/CD complète avec runner distant, mais j’ai préparé une procédure automatisable et réutilisable sous forme de script de déploiement.
Cette approche répond au besoin de :
déployer rapidement ;
limiter les erreurs manuelles ;
garantir une séquence identique à chaque mise en production.

1.2 Diagramme du pipeline

[ Développeur ]
      |
      v
[ Dépôt Git ]
      |
      v
[ Déclenchement manuel ]
      |
      v
[ Serveur de production ]
  - récupération du code
  - composer install
  - configuration Laravel
  - migrations
  - clear caches
      |
      v
[ Smoke tests ]
  - HTTP 200
  - route principale
  - API
  - webhook

2. Outillage retenu

Outil	Rôle dans le pipeline	Justification
Git	versionnement et récupération du code	standard de fait pour le suivi des versions
SSH	accès sécurisé au serveur	permet l’exécution distante sécurisée
Bash	automatisation des commandes de déploiement	simple, natif, rapide à mettre en place
Composer	installation des dépendances PHP	indispensable pour Laravel
Artisan	migrations, nettoyage du cache, optimisation	outil standard Laravel
Apache	exposition de l’application après mise à jour	serveur web cible de production
curl	smoke tests post-déploiement	permet de valider rapidement l’accessibilité

3. Déclenchement du déploiement

3.1 Mode de déclenchement

Le déclenchement est manuel.
Dans le contexte de l’évaluation, cela permet :
de garder le contrôle sur la séquence ;
d’exécuter le déploiement uniquement après validation ;
d’éviter un déploiement automatique non contrôlé sur un environnement encore en phase de correction.
Le déclenchement se fait par exécution d’une suite de commandes ou d’un script dédié sur le serveur.

3.2 Reproductibilité

La procédure de déploiement est reproductible car elle repose sur :
un ordre de commandes fixe ;
un emplacement stable du projet (/var/www/opstrack) ;
un fichier .env configuré ;
les migrations Laravel ;
le nettoyage systématique des caches.
La même séquence peut être rejouée après une mise à jour du code ou après une restauration.

4. Contrôles préalables au déploiement

Contrôle	Description	Critère de passage
Intégrité du code	présence des fichiers du projet Laravel	projet complet présent
Dépendances PHP	exécution de composer install	pas d’erreur bloquante
Compatibilité environnement	extensions PHP requises disponibles	environnement compatible
Base de données	accès MySQL opérationnel	connexion OK
Configuration Laravel	fichier .env présent et valide	APP_KEY, DB et secrets configurés
Apache	configuration VirtualHost valide	apache2ctl configtest retourne Syntax OK

5. Mise à jour de la production

La mise à jour de la production suit les étapes suivantes :
cd /var/www/opstrack
composer install --no-dev --optimize-autoloader
php artisan optimize:clear
php artisan migrate --force
sudo systemctl reload apache2
Dans le cadre de cette mise en production, des corrections ont également été appliquées directement sur le code :
correction d’une migration mal ordonnancée ;
correction du contrôleur de recherche tickets ;
correction du webhook ;
correction de l’invalidation du cache dashboard.

6. Vérification post-déploiement

6.1 Smoke tests

Test	Commande ou méthode	Résultat attendu
Application principale accessible	curl -I http://localhost	HTTP 200
Domaine public accessible	curl -I http://eval-dfs-p-tpl-20262-01.it-students.fr	HTTP 200
Page d’accueil Laravel	navigateur / navigation privée	affichage du dashboard
API tickets protégée	curl sans token	HTTP 401
API tickets valide avec token	curl avec Bearer token	réponse JSON 200
Validation entrée mal formée	priority[]=high avec Accept JSON	HTTP 422
Webhook sécurisé	curl -u user:password	réponse JSON 200 après correction
Mise à jour métier webhook	vérification SQL du ticket	statut mis à jour

6.2 Preuve de déploiement réussi

Les preuves observées après déploiement sont les suivantes :
l’application répond en HTTP 200 ;
le VirtualHost Apache pointe correctement sur /var/www/opstrack/public ;
les migrations et le seeding ont été exécutés avec succès ;
l’API /api/v1/tickets répond correctement ;
les erreurs 500 identifiées pendant l’épreuve ont été corrigées ;
le webhook fonctionne après correction du point d’entrée Laravel et de l’exemption CSRF ;
le ticket ciblé par le webhook est bien mis à jour en base.

7. Conduite à tenir en cas d’échec

En cas d’échec de déploiement, la procédure retenue est la suivante :
consulter les logs Apache et Laravel ;
identifier si l’échec vient :
du code ;
d’une dépendance ;
de la base de données ;
de la configuration ;
interrompre la séquence avant ouverture au public ;
corriger l’erreur ;
relancer les étapes concernées ;
rejouer les smoke tests.
Exemples d’échecs réellement rencontrés pendant l’épreuve :
incompatibilité de version ext-mongodb ;
erreur de migration liée à l’ordre des tables ;
erreur 500 sur le webhook ;
erreur 500 puis 302 sur l’API tickets en cas d’entrée mal formée.

8. Scripts et fichiers de configuration

Fichier	Rôle
/var/www/opstrack/.env	configuration d’environnement de production
/etc/apache2/sites-available/000-default.conf	configuration Apache de l’application
/var/www/opstrack/bootstrap/app.php	configuration middleware et exemption CSRF webhook
/var/www/opstrack/routes/web.php	routes web, dont le point d’entrée webhook
/var/www/opstrack/public/hooks.php	point d’entrée public du webhook
/var/www/opstrack/app/Http/Controllers/Api/TicketController.php	logique API tickets
/var/www/opstrack/app/Http/Controllers/WebhookController.php
