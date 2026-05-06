# Source brute — Erreur HTTP vs HTTPS Angular
Date    : 2026-05-06
Projet  : task-manager
Erreur  : ERR_CONNECTION_REFUSED sur http://localhost:7063
Contexte: Angular appelle http:// mais API tourne sur https://
Cause   : launchSettings.json configure HTTPS sur 7063
          environment.ts utilisait http:// au lieu de https://
Fix     : Changer environment.ts : apiUrl = "https://localhost:7063"
