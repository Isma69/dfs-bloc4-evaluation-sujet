Documentation d’API

Vue d’ensemble de l’API
Champ Valeur URL de base http://localhost/api/v1 Format JSON Authentification Bearer Token (header Authorization) L’API permet de consulter et filtrer les tickets ainsi que d’interagir indirectement avec les interventions via un webhook externe.

Endpoints disponibles
Méthode Endpoint Description Authentification requise GET /tickets Liste paginée des tickets Oui GET /tickets?search=... Recherche par titre ou référence Oui GET /tickets?priority=... Filtre par priorité Oui GET /tickets/{id} Détail d’un ticket Oui POST /tickets Création d’un ticket Oui PUT/PATCH /tickets/{id} Mise à jour d’un ticket Oui POST /hooks.php Webhook externe (création intervention + update ticket) Oui (Basic Auth)

Exemples de requêtes et réponses
🔹 Récupérer tous les tickets curl -H "Authorization: Bearer super_secure_token_2026" 
http://localhost/api/v1/tickets Réponse : { "data": [ { "id": 1, "reference": "INC-240301", "title": "Intermittent payment terminal outage", "priority": "critical", "status": "in_progress" } ] } 🔹 Filtrer par priorité curl -H "Authorization: Bearer super_secure_token_2026" 
"http://localhost/api/v1/tickets?priority=high" 🔹 Requête invalide (validation) curl -H "Authorization: Bearer super_secure_token_2026" 
-H "Accept: application/json" 
"http://localhost/api/v1/tickets?priority[]=high" Réponse : { "message": "validation.in", "errors": { "priority": ["validation.in"] } } 🔹 Webhook (mise à jour ticket) curl -u secure_user:VeryStrongPassword123! 
-H "Accept: application/json" 
-d "ticket_reference=INC-240302" 
-d "status=resolved" 
-d "summary=Resolved by external system" 
http://localhost/hooks.php Réponse : { "message": "Webhook processed.", "intervention_id": 3 }

Codes d’erreur
Code Signification 200 Requête réussie 401 Authentification requise ou invalide 419 CSRF token manquant ou invalide 422 Données invalides (validation Laravel) 500 Erreur serveur interne
