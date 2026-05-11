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
