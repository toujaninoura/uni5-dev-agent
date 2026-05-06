# Source brute — Bootstrap non charge Angular
Date    : 2026-05-06
Projet  : task-manager
Erreur  : Classes Bootstrap sans effet visuel
Contexte: npm install bootstrap fait mais styles non appliques
Cause   : angular.json ne contenait pas Bootstrap dans styles et scripts
Fix     : Ajouter dans angular.json :
  "styles": ["node_modules/bootstrap/dist/css/bootstrap.min.css", "src/styles.css"]
  "scripts": ["node_modules/bootstrap/dist/js/bootstrap.bundle.min.js"]
