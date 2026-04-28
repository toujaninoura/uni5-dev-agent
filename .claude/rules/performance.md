# Rules - Performance

## .NET Performance
- AsNoTracking() sur TOUTES les requetes lecture
- Select() pour ne charger que les colonnes necessaires
- Include() explicite uniquement si relation necessaire
- Pagination obligatoire sur toutes les listes
- Index sur colonnes filtrees frequemment
- Eviter N+1 queries -> utiliser Include ou projection

## Async .NET
- Async/await sur tous les I/O
- Jamais de .Result ou .Wait() -> deadlock
- CancellationToken sur les operations longues
- ConfigureAwait(false) dans les librairies

## EF Core Performance
- Compiler les queries frequentes : EF.CompileAsyncQuery
- Utiliser des projections directes en DTO quand possible
- Eviter de charger des entites juste pour les supprimer
- BulkInsert pour les insertions massives

## Angular Performance
- OnPush ChangeDetection sur tous les composants
- TrackBy sur tous les *ngFor
- Lazy loading sur toutes les features
- Async pipe au lieu de subscribe
- Eviter les calculs dans les templates -> pipes custom
- Debounce sur les champs de recherche : 300ms

## Images et Assets Angular
- Lazy loading sur les images : loading="lazy"
- Optimiser les images avant build
- ng build --configuration production -> tree shaking auto

## Cache
- HttpClient cache sur les donnees statiques
- IMemoryCache sur les donnees peu changeantes cote .NET
- localStorage pour les preferences utilisateur Angular
