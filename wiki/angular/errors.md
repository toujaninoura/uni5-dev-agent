# Wiki Angular — Erreurs connues et solutions

## Bootstrap non charge
**Erreur** : Classes Bootstrap sans effet
**Cause**  : Bootstrap absent de angular.json styles/scripts
**Fix**    : Ajouter dans angular.json :
  styles: ["node_modules/bootstrap/dist/css/bootstrap.min.css"]
  scripts: ["node_modules/bootstrap/dist/js/bootstrap.bundle.min.js"]
**Prevention** : Verifier angular.json apres npm install bootstrap

## Subscribe sans unsubscribe
**Erreur** : Memory leak dans les composants
**Cause**  : subscribe() dans ngOnInit sans cleanup
**Fix**    : Utiliser async pipe dans le template
**Prevention** : Jamais de subscribe() — toujours async pipe

## JWT Interceptor non enregistre
**Erreur** : 401 Unauthorized sur tous les appels API
**Cause**  : Interceptor absent de app.config.ts
**Fix**    : Ajouter dans provideHttpClient(withInterceptors([jwtInterceptor]))
**Prevention** : Verifier app.config.ts apres creation de l interceptor

## CORS — Provisional headers
**Erreur** : Provisional headers are shown
**Cause**  : API bloque la requete Angular (CORS ou API eteinte)
**Fix**    : Verifier que API tourne + CORS configure pour http://localhost:4200
**Prevention** : Toujours demarrer l API avant Angular
