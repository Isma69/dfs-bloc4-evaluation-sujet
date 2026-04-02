Journal de sécurité
Ce document recense les failles de sécurité identifiées pendant l’épreuve, leur évaluation et les mesures correctives appliquées.
Faille 1
Champ	Description
Date de détection	2 avril 2026
Composant concerné	Webhook (hooks.php)
Description de la faille	Le webhook était exposé via un fichier PHP public exécuté hors du contexte complet de Laravel. Cela entraînait une mauvaise initialisation de l’application et l’absence de certains mécanismes de sécurité (gestion correcte des dépendances, configuration, middleware).
Sévérité estimée	Haute
Impact potentiel	- exécution incorrecte du code métier
- contournement des mécanismes de sécurité Laravel
- instabilité du service (erreurs 500)
- risque d’exploitation si logique étendue
Mesure corrective appliquée	- intégration du webhook dans le cycle Laravel (via contrôleur)
- suppression de la logique isolée
- utilisation correcte de config() et des services Laravel
- sécurisation du point d’entrée
Statut	Corrigé
Preuve de correction	- disparition de l’erreur Class "config" does not exist
- réponse JSON valide du webhook
- création d’intervention en base via appel curl fonctionnel


Faille 2
Champ	Description
Date de détection	2 avril 2026
Composant concerné	Endpoint webhook
Description de la faille	Absence initiale de protection CSRF et gestion incomplète des requêtes POST externes, entraînant un blocage (erreur 419) ou une exposition potentielle si mal configuré.
Sévérité estimée	Moyenne
Impact potentiel	- rejet des requêtes légitimes (webhook inutilisable)
- possibilité de contournement si désactivation globale du CSRF
- incohérence du comportement API
Mesure corrective appliquée	- mise en place d’une exemption CSRF ciblée pour le webhook
- ajout d’une authentification Basic Auth
- validation stricte des données entrantes (validate())
Statut	Corrigé
Preuve de correction	- disparition de l’erreur CSRF token mismatch
- appel curl avec authentification réussi
- mise à jour effective du ticket en base

Faille 3
Champ	Description
Date de détection	2 avril 2026
Composant concerné	API /api/v1/tickets
Description de la faille	L’API acceptait des paramètres mal formés (ex : priority[]=high) entraînant une erreur serveur (500) au lieu d’une réponse contrôlée. Cela révèle un manque de validation stricte des entrées utilisateur.
Sévérité estimée	Moyenne
Impact potentiel	- plantage de l’application (DoS applicatif léger)
- exposition de comportements internes
- surface d’attaque élargie (input mal contrôlé)
Mesure corrective appliquée	- validation stricte des paramètres (string attendu)
- gestion correcte des erreurs avec retour HTTP 422
- ajout du header Accept: application/json pour forcer le format API
Statut	Corrigé
Preuve de correction	- requête invalide retourne désormais 422 Unprocessable Entity
- plus aucune erreur 500 sur input mal formé


Faille 4
Champ	Description
Date de détection	2 avril 2026
Composant concerné	Base de données / migrations
Description de la faille	Mauvais ordre d’exécution des migrations entraînant une erreur SQL sur clé étrangère (Failed to open the referenced table 'tickets'). Cela peut bloquer le déploiement et rendre l’application indisponible.
Sévérité estimée	Haute
Impact potentiel	- impossibilité de déployer l’application
- indisponibilité complète du service
- incohérence du schéma de base de données
Mesure corrective appliquée	- correction de l’ordre des migrations
- vérification de la dépendance entre tables
- relance complète des migrations
Statut	Corrigé
Preuve de correction	- migrations exécutées sans erreur
- création d’interventions fonctionnelle via Tinker et webhook
