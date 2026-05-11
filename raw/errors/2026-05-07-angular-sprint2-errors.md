# Angular Sprint 2 — Erreurs rencontrées lors du développement de la feature task sharing

> Source: internal — dev session 2026-05-07 (issues #31 #32 #33 #34 #39 #40)
> Collected: 2026-05-07
> Published: 2026-05-07

## ERROR 1 — URL API hardcodée dans les services Angular

Erreur : En production, l'URL https://localhost:7063 dans task.service.ts et task-sharing.service.ts casse tous les appels API hors de la machine du développeur.
Cause : URL hardcodée directement dans la propriété private readonly API au lieu d'utiliser environment.apiUrl.
Fix : Utiliser `${environment.apiUrl}/api/v1/tasks` et importer environment depuis src/environments/environment.
Prévention : Toujours construire les URLs de service depuis environment.apiUrl, jamais de localhost dans le code source.

## ERROR 2 — Routes incorrectes dans TaskSharingService

Erreur : 404 sur getPendingInvitations, acceptInvitation, rejectInvitation.
Cause : Routes construites depuis les noms de méthodes Angular au lieu des endpoints backend réels : /invitations/pending au lieu de /shared/pending, /invitations/accept au lieu de /collaborators/accept, /invitations/reject au lieu de /collaborators/reject.
Fix : Aligner les routes Angular sur les routes backend définies dans le controller C# : GET /shared/pending, POST /{id}/collaborators/accept, POST /{id}/collaborators/reject.
Prévention : Toujours vérifier les routes dans le controller backend avant d'écrire les méthodes du service Angular. Créer les specs avec HttpTestingController — le test aurait révélé le mismatch immédiatement.

## ERROR 3 — (window as any).bootstrap.Modal dans un composant Angular

Erreur : Manipulation DOM directe + type any. Code non testable en Jasmine, viole TypeScript strict mode.
Cause : Appel natif Bootstrap via document.getElementById + new (window as any).bootstrap.Modal(el).show() pour ouvrir un modal.
Fix : Remplacer par une variable booléenne showShareModal = false, *ngIf sur le composant modal, et @Output() modalClosed pour fermer.
Prévention : Jamais de window ou document dans un composant Angular. Contrôler les modals par état (boolean) + bindings Angular.

## ERROR 4 — Observable switchMap qui se termine sur erreur (forkJoin sans catchError interne)

Erreur : Après une erreur réseau (401, 500), loadData() devient silencieusement inopérant — les appels suivants à loadTrigger$.next() sont ignorés.
Cause : Le forkJoin est dans un switchMap souscrit une seule fois dans ngOnInit. Si forkJoin émet une erreur sans catchError, l'erreur remonte et termine définitivement la souscription au Subject.
Fix : Encapsuler le forkJoin dans un .pipe(catchError(() => EMPTY)) à l'intérieur du switchMap, pas à l'extérieur. Le flux principal reste vivant.
Prévention : Toute souscription longue durée (Subject + switchMap dans ngOnInit) doit avoir un catchError interne sur chaque opération interne.

## ERROR 5 — getUserId() ne vérifie pas l'expiration du token JWT

Erreur : getUserId() retourne un userId valide même pour un token expiré — bouton "Partager" visible alors que les appels API retourneront 401.
Cause : Le payload JWT est décodé (atob) sans vérifier le champ exp.
Fix : Après décodage, comparer payload.exp avec Math.floor(Date.now() / 1000). Si expiré, retourner 0 ou null.
Prévention : Toujours vérifier exp après décodage JWT côté Angular. Le JWT interceptor gère les 401 mais n'empêche pas l'affichage incorrect de l'UI avant l'appel.
