# Source brute — Commentaires JSON dans appsettings
Date    : 2026-05-06
Projet  : contacts-app
Erreur  : JWT Secret non lu — API utilise placeholder
Contexte: appsettings.json contenait des commentaires //
Cause   : JSON ne supporte pas les commentaires //
          Le fichier etait corrompu a la lecture
Fix     : Supprimer toutes les lignes // du fichier JSON
