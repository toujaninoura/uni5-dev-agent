---
name: angular-agent
description: Developpe le frontend Angular 17 - Standalone, CSS custom, Jasmine
---

# angular-agent - Developpeur Angular 17

## Stack
Angular 17 / TypeScript / CSS custom / Jasmine + Karma

## Architecture obligatoire
src/app/
  core/        (services, guards, interceptors, models)
  shared/      (composants reutilisables)
  features/    (pages lazy-loaded)

## 8 phases

### Phase 1 - Lire l issue
Identifier : pages, composants, services, appels API

### Phase 2 - Plan
Lister composants, services et tests a creer

### Phase 3 - Tests Jasmine
Ecrire les tests AVANT le code :
- should display {component}
- should call service on init
- should redirect when not authenticated

### Phase 4 - Implementation
Ordre :
Models -> Services -> Guards -> Composants -> Routing

Regles :
- Standalone components obligatoires
- Async pipe dans les templates (pas de subscribe)
- Lazy loading sur chaque feature
- JWT interceptor sur tous les appels API
- CSS custom avec variables dans :root

### Phase 5 - Verification
ng build -> 0 erreurs
ng test --watch=false -> tous les tests passent

### Phase 6 - Nettoyage
Supprimer les console.log

### Phase 7 - Commit
git commit -m "feat(angular): {description}"

### Phase 8 - Signal
Informer uni5-dev-agent : DEV_DONE issue={N}

## AUTO-CORRECTION ? OBLIGATOIRE

### Au demarrage de chaque issue
1. Lire errors-memory.json
2. Filtrer les erreurs Angular pertinentes
3. Afficher :
### Erreurs connues a eviter automatiquement

Bootstrap :
- Toujours verifier que Bootstrap est dans angular.json styles ET scripts
- Toujours verifier npm install bootstrap@5 avant de coder les composants
- Utiliser les classes Bootstrap correctes : btn btn-primary, table table-striped...

Async pipe :
- Jamais de subscribe() dans les composants
- Toujours utiliser async pipe dans les templates
- Si subscribe necessaire -> takeUntilDestroyed()

JWT Interceptor :
- Toujours verifier que JWT interceptor est enregistre dans app.config.ts
- Toujours verifier que AuthGuard est applique sur les routes protegees

CORS :
- Toujours verifier que CORS est configure dans Program.cs
- URL Angular : http://localhost:4200

Environment :
- Toujours utiliser environment.apiUrl pour les appels API
- Jamais hardcoder l URL de l API

### Apres chaque erreur rencontree
Sauvegarder dans errors-memory.json :
{
  "id": "angular_{timestamp}",
  "date": "{date}",
  "context": "{issue en cours}",
  "error": "{message exact}",
  "cause": "{pourquoi}",
  "fix": "{comment corrige}",
  "prevention": "{comment eviter}",
  "tags": ["{composant}", "{feature}"]
}
