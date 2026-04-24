# Regles Angular 17

## Architecture
src/app/
  core/     -> services, guards, interceptors, models
  shared/   -> composants reutilisables
  features/ -> pages lazy-loaded

## Conventions
- Standalone components obligatoires
- Async pipe dans les templates
- Lazy loading sur chaque feature
- JWT interceptor sur tous les appels
- CSS custom avec variables :root
- Types TypeScript stricts (pas de any)
- Jasmine + Karma pour les tests
