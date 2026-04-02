Changelog

Toutes les modifications notables apportées pendant l’épreuve sont documentées dans ce fichier.

Le format s’inspire de Keep a Changelog.

[Session du 2 avril 2026]

Ajout

Mise en place de l’environnement de production (Apache, PHP, MySQL, MongoDB)

Déploiement initial de l’application Laravel

Configuration du VirtualHost Apache pointant vers /public

Implémentation de l’accès sécurisé à l’API via Bearer Token

Mise en place du webhook avec authentification Basic Auth

Ajout du service de journalisation métier (EventLogService)

Mise en place des tests de validation via curl (API, webhook)

Création d’interventions via webhook (intégration métier)

Modifié

Configuration Apache pour exposer correctement l’application Laravel

Ajustement des headers HTTP pour forcer les réponses JSON (Accept: application/json)

Amélioration de la gestion des filtres dans l’API tickets (search, priority)

Mise à jour du contrôleur webhook pour respecter le fonctionnement Laravel

Adaptation de la configuration middleware pour permettre le fonctionnement du webhook

Nettoyage et optimisation des caches Laravel (optimize:clear)

Corrigé

Erreur 500 liée à une mauvaise initialisation du framework Laravel dans hooks.php

Erreur Class "config" does not exist corrigée en utilisant correctement le contexte Laravel

Erreur de migration :

clé étrangère sur interventions.ticket_id

causée par un ordre incorrect de création des tables

Erreur HTTP 302 sur API corrigée via gestion du header Accept

Validation incorrecte des paramètres (priority[]) désormais gérée avec retour HTTP 422

Correction du comportement de recherche avec caractères spéciaux (search=')

Correction de l’invalidation du cache dashboard

Sécurité

Mise en place de l’authentification API via Bearer Token

Sécurisation du webhook via Basic Auth

Ajout d’une validation stricte des entrées utilisateur

Correction de l’exposition du webhook hors contexte Laravel

Mise en place de l’exemption CSRF ciblée pour le webhook

Protection des accès sensibles via fichier .env
Restriction des accès serveur (SSH, firewall)
