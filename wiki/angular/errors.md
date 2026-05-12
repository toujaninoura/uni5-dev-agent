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

## URL API hardcodee dans les services Angular
**Erreur**     : En production, https://localhost:7063 dans task.service.ts/task-sharing.service.ts casse tous les appels API hors de la machine du developpeur
**Cause**      : `private readonly API = 'https://localhost:7063/api/v1/...'` hardcode au lieu d utiliser environment.apiUrl
**Fix**        : `` private readonly API = `${environment.apiUrl}/api/v1/tasks` `` — importer environment depuis src/environments/environment
**Prevention** : Jamais de localhost dans le code source. Toujours construire les URLs depuis environment.apiUrl

## Routes incorrectes dans un service Angular (mismatch backend)
**Erreur**     : 404 sur getPendingInvitations, acceptInvitation, rejectInvitation
**Cause**      : Routes construites depuis les noms de methodes Angular plutot que depuis les endpoints backend reels (/invitations/pending au lieu de /shared/pending, /invitations/accept au lieu de /collaborators/accept)
**Fix**        : Aligner les routes Angular sur les routes du controller C# : GET /shared/pending, POST /{id}/collaborators/accept, POST /{id}/collaborators/reject
**Prevention** : Toujours lire le controller backend avant d ecrire le service Angular. Ecrire les specs avec HttpTestingController — le test aurait revele le mismatch immediatement

## (window as any).bootstrap.Modal dans un composant Angular
**Erreur**     : Manipulation DOM directe + type any. Code non testable en Jasmine, viole TypeScript strict mode
**Cause**      : `document.getElementById` + `new (window as any).bootstrap.Modal(el).show()` pour ouvrir un modal Bootstrap
**Fix**        : Remplacer par `showShareModal = false` (boolean), `*ngIf` sur le composant modal, `@Output() modalClosed` pour fermer
**Prevention** : Jamais de window ou document dans un composant Angular. Controler les modals par etat (boolean) + bindings Angular

## Observable switchMap qui se termine sur erreur (forkJoin sans catchError interne)
**Erreur**     : Apres une erreur reseau (401, 500), loadData() devient silencieusement inoperant — les appels suivants a loadTrigger$.next() sont ignores
**Cause**      : Le forkJoin est dans un switchMap souscrit une seule fois dans ngOnInit. Si forkJoin emet une erreur sans catchError interne, l erreur remonte et termine definitivement la souscription au Subject
**Fix**        : `forkJoin({...}).pipe(catchError(() => EMPTY))` a l interieur du switchMap — pas a l exterieur. Le flux principal reste vivant
**Prevention** : Toute souscription longue duree (Subject + switchMap dans ngOnInit) doit avoir un catchError interne sur chaque operation forkJoin/mergeMap interne

## getUserId() sans verification d expiration du token JWT
**Erreur**     : getUserId() retourne un userId valide meme pour un token expire — UI affiche des actions (bouton "Partager") alors que les appels API retourneront 401
**Cause**      : Payload JWT decode via atob sans verifier le champ exp
**Fix**        : `if (payload.exp && payload.exp < Math.floor(Date.now() / 1000)) return 0;` apres le decodage
**Prevention** : Toujours verifier exp apres decodage JWT cote Angular. Le JWT interceptor gere les 401 mais n empeche pas l affichage incorrect de l UI avant l appel

## RouterTestingModule deprecated — TypeError reading 'root'
**Erreur** : `TypeError: Cannot read properties of undefined (reading 'root')` dans les specs Jasmine quand le composant utilise `RouterLink`
**Cause**  : `RouterTestingModule` (deprecated Angular 16+) conflicte avec `{ provide: Router, useValue: routerSpy }` — le spy n'a pas la propriete `root` attendue par `RouterLink`
**Fix**    : Remplacer `RouterTestingModule.withRoutes([])` par `provideRouter([]) + provideLocationMocks()`. Ne pas fournir de spy Router — injecter le vrai puis `spyOn(TestBed.inject(Router), 'navigate')`
**Prevention** : Jamais `RouterTestingModule` en Angular 17. Pattern : `provideRouter([])` + `provideLocationMocks()` + `spyOn(router, 'navigate')`

## TS2345 — signature mismatch apres changement de type sur un service
**Erreur** : Build echoue `TS2345 Argument of type { email, password } is not assignable to parameter of type RegisterRequest`
**Cause**  : Signature de service changee sans mettre a jour tous les appelants (tests, composants)
**Fix**    : Chercher tous les appelants avec grep avant de changer une signature. Mettre a jour signature + tous les appelants dans un seul commit
**Prevention** : `grep -r "serviceMethod" src/` avant tout changement de signature. Ne jamais changer une signature sans scanner les usages

## isLoading non remis a false apres succes dans un composant
**Erreur** : Bouton submit reste desactive apres une requete reussie
**Cause**  : `this.isLoading = false` absent du callback `next` — uniquement dans `error`
**Fix**    : Toujours reset dans les deux callbacks : `next: () => { this.isLoading = false; ... }` et `error: () => { this.isLoading = false; ... }`
**Prevention** : Pattern obligatoire — `isLoading = false` dans `next` ET `error`. Ne jamais supposer que la navigation detruira le composant

## npm install bloque — UNABLE_TO_VERIFY_LEAF_SIGNATURE
**Erreur**     : `npm install` bloque pendant 20+ minutes avec `UNABLE_TO_VERIFY_LEAF_SIGNATURE` et `ENOTFOUND`, chaque package prenant 21-70 secondes
**Cause**      : `npm strict-ssl=true` provoque des échecs SSL contre registry.npmjs.org. npm retente 3 fois par package avant le cache → lenteur extrême
**Fix**        : `npm config set strict-ssl false` puis `npm install --no-audit --no-fund`. Tuer les process npm concurrents avant de relancer.
**Prevention** : Au démarrage d un projet Angular : vérifier `npm config get strict-ssl` — si true, set false. Jamais lancer plusieurs npm install en parallèle sur le même répertoire.

## ng new en background bloque les npm install concurrents
**Erreur**     : `ng new` lancé en background + `npm install` manuel concurrent — aucun ne se termine
**Cause**      : `ng new` lance `npm install` en interne. Un npm install externe en parallèle crée une race condition — les deux se bloquent mutuellement.
**Fix**        : Attendre la fin complète de `ng new` avant tout `npm install` supplémentaire. Ou tuer tous les process `node.exe` concurrents.
**Prevention** : Après `ng new` : attendre la fin complète, vérifier que `node_modules` existe, seulement ensuite installer les packages supplémentaires.

## ng new génère Angular 21 au lieu d Angular 17 (Vitest au lieu de Jasmine)
**Erreur**     : `ng new` génère un projet Angular 21 avec Vitest par défaut au lieu de Jasmine+Karma
**Cause**      : Angular CLI 21.x installé globalement — `ng new` crée toujours un projet correspondant à la version du CLI installé
**Fix**        : Adapter les specs pour Vitest : `vi.fn()`, `vi.spyOn()` au lieu de `jasmine.createSpyObj`. Utiliser `provideRouter([])`, `provideHttpClient()` + `provideHttpClientTesting()` (pas les anciens modules deprecated).
**Prevention** : Vérifier `ng version` avant d écrire les tests. Si Angular 21+ : utiliser les globals Vitest configurés dans `tsconfig.spec.json`. Utiliser les functional providers, pas les module imports deprecated.

## See Also
- [Architecture Angular TaskManager — composants, routes, services](./architecture.md)
