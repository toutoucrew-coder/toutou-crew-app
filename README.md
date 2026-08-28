# Toutou Crew — Outil de gestion financière

Deux versions du même outil, chacune un fichier HTML autonome (structure, CSS, JS tout inclus dedans — aucune dépendance externe à installer).

## toutou-crew-finances.html
Version actuelle, pensée pour tourner comme artefact Claude (claude.ai). Utilise `window.storage`
(API de stockage persistant propre aux artefacts Claude, disponible sur les plans payants Pro/Max/Team/Enterprise).
Si `window.storage` n'existe pas (ouverture en dehors de Claude), elle bascule automatiquement sur le
`localStorage` du navigateur (données propres à cet appareil/navigateur, non synchronisées).

## toutou-crew-drive.html
Nouvelle version en cours, pensée pour un hébergement autonome (hors Claude), avec les données stockées
sur Google Drive de l'utilisateur (accessible depuis n'importe quel appareil connecté au même compte
Google), sans besoin d'abonnement Claude payant.

### À FAIRE avant que cette version fonctionne :
1. Créer un projet Google Cloud + activer l'API Google Drive + configurer l'écran de consentement OAuth
   + créer un identifiant "ID client OAuth" (type Application Web), avec pour "Origines JavaScript
   autorisées" l'URL exacte où ce fichier sera hébergé (prévu : https://app.toutoucrew.fr).
2. Dans le fichier, remplacer la constante en haut du script :
   ```js
   const GOOGLE_CLIENT_ID = 'REMPLACE_MOI.apps.googleusercontent.com';
   ```
   par le vrai Client ID obtenu à l'étape 1.
3. Héberger ce fichier (renommé index.html) sur app.toutoucrew.fr — projet en cours : domaine chez OVH,
   hébergement du site principal sur GitHub Pages. Comme le custom domain "toutoucrew.fr" est déjà pris
   par le dépôt du site principal sur GitHub Pages, il faut créer un DEUXIÈME dépôt GitHub dédié
   (ex: toutou-crew-app), y activer GitHub Pages avec app.toutoucrew.fr comme custom domain, et ajouter
   côté OVH (zone DNS de toutoucrew.fr, PAS la section hébergement) un enregistrement CNAME :
   sous-domaine "app" → cible "<identifiant-github>.github.io." (avec le point final).
4. Activer le HTTPS (Enforce HTTPS) dans les réglages Pages du nouveau dépôt une fois le DNS propagé.

### Comment fonctionne le stockage Drive (déjà implémenté et testé)
- `window.storage.get/set/delete/list` a été entièrement réimplémenté pour parler à l'API Google Drive
  (au lieu du localStorage/window.storage Claude) — le reste de l'app (toute la logique métier, le
  rendu, les interactions) n'a PAS changé, seule cette couche a été remplacée.
- Toutes les données sont stockées dans UN SEUL fichier JSON sur le Drive de l'utilisateur, nommé
  `toutou-crew-data.json`, contenant les mêmes clés qu'avant (`clients`, `entries`, `settings`).
- Scope OAuth utilisé : `https://www.googleapis.com/auth/drive.file` (accès limité aux fichiers créés
  par l'app elle-même, pas à tout le Drive — plus sûr).
- Au chargement : recherche du fichier existant par nom (`files.list`) ; s'il existe, téléchargement +
  parsing (`files.get?alt=media`) ; sinon, création d'un fichier vide (`files.create`) puis upload
  initial.
- À chaque sauvegarde : re-upload du JSON complet (`files.update` via `uploadType=media`, PATCH).
- Testé avec un mock de l'API Drive/Google Identity Services (voir les tests dans l'historique de
  conversation) : création initiale, sauvegarde, et relecture depuis une "session" différente (simulant
  un autre appareil) fonctionnent correctement.

### Limite connue
Pas de gestion de conflit si l'app est utilisée sur deux appareils EN MÊME TEMPS (dernier enregistrement
écrase le précédent). Risque faible pour un usage solo, mais bon à savoir si ça évolue vers du multi-
utilisateur.

## Contexte général du projet
Micro-entreprise de promenade/éducation canine à Paris 19e (Merlin). L'outil suit : encaissements,
clients (avec 1-2 chiens par foyer), abonnements récurrents (plannings par jour de semaine), balades
test ponctuelles, clôture mensuelle avec renouvellement, calculs URSSAF/ACRE/ARE/plafond micro-
entreprise. Design : fond `#f2e9d8`, texte `#44413a`, accent `#4a9cf1`, montants oranges `#e6743f`.
