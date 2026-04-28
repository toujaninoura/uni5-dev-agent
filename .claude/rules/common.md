# Règles communes

## Code
- Immutabilité : toujours créer de nouveaux objets
- Fichiers : 200-400 lignes, 800 max
- Erreurs : gérées à chaque niveau, jamais silencieuses
- Validation : toutes les entrées à la frontière du système

## Sécurité
- Jamais de secrets dans le code
- .env dans .gitignore, .env.example committé
- SQL : requêtes paramétrées uniquement

## Git
- Commits conventionnels : type(scope): description
- Un commit = une responsabilité
- Jamais de --no-verify ni force push sur main

## Tests
- TDD : tests avant le code
- Couverture minimale : 80%
- Nommage : should {comportement} when {condition}
