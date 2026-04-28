# Rules - Common

## Code
- Immutabilite : toujours creer de nouveaux objets, ne jamais muter
- Fichiers : 200-400 lignes ideal, 800 max absolu
- Organisation : par feature/domaine, pas par type
- Erreurs : gerees a chaque niveau, jamais silencieuses
- Validation : toutes les entrees a la frontiere du systeme
- Methodes : max 20 lignes
- Pas de magic numbers -> constantes nommees
- Pas de code commente -> Git garde l historique

## Securite
- Jamais de secrets dans le code
- .env dans .gitignore, .env.example committe
- SQL : requetes parametrees uniquement
- Jamais exposer la stack trace en production

## Nommage
- Variables et fonctions : noms explicites sans abreviations obscures
- Un niveau d abstraction par methode
- Nommer les choses selon ce qu elles font, pas comment elles le font
