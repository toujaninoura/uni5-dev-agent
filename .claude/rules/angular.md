# Rules - Angular 17

## Stack obligatoire
- Angular 17+ standalone components
- TypeScript strict mode
- CSS custom (pas de librairie UI)
- Jasmine + Karma pour les tests

## Architecture obligatoire
src/app/
  core/
    services/      <- services globaux singleton
    guards/        <- AuthGuard, RoleGuard
    interceptors/  <- JWT, Error
    models/        <- interfaces TypeScript
  shared/
    components/    <- composants reutilisables
    directives/    <- directives custom
    pipes/         <- pipes custom
  features/
    auth/          <- login, register
    {feature}/     <- une feature = un module lazy

## Regles obligatoires
- Standalone components uniquement
- Async pipe dans les templates (jamais subscribe dans .ts)
- Lazy loading sur chaque feature module
- JWT interceptor sur tous les appels API
- AuthGuard sur toutes les routes protegees
- Interfaces TypeScript sur tous les modeles (jamais any)
- CSS isole dans chaque composant
- Mobile-first avec breakpoints : 576px 768px 992px 1200px

## Interdit
- NgModules sauf AppModule
- Subscribe sans unsubscribe
- Type any
- Logique metier dans les composants
- Style inline dans les templates
- console.log en production

## Commandes
- Dev  : ng serve --proxy-config proxy.conf.json
- Build: ng build --configuration production
- Test : ng test --watch=false --code-coverage
- Lint : ng lint
