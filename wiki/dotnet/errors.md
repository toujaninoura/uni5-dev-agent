# Wiki .NET — Erreurs connues et solutions

## JWT Secret trop court
**Erreur** : IDX10720 key size must be greater than 256 bits
**Cause**  : Secret JWT < 32 caracteres dans appsettings.json
**Fix**    : dotnet user-secrets set "JWT:Secret" "{32+ chars}"
**Prevention** : Toujours generer avec GUID double :
  [System.Guid]::NewGuid().ToString("N") + [System.Guid]::NewGuid().ToString("N")

## Commentaires dans appsettings.json
**Erreur** : JSON parse error
**Cause**  : // commentaires dans fichiers .json
**Fix**    : Supprimer tous les commentaires //
**Prevention** : Jamais de // dans les fichiers JSON

## CORS non configure
**Erreur** : Provisional headers are shown / ERR_CONNECTION_REFUSED
**Cause**  : AddCors ou UseCors manquant dans Program.cs
**Fix**    : Ajouter AddCors + UseCors("AngularApp") dans Program.cs
**Prevention** : Toujours verifier CORS dans Program.cs avant dotnet run

## Deux policies CORS en conflit
**Erreur** : CORS bloque malgre configuration
**Cause**  : AddCorsPolicy() ET AddCors() en meme temps
**Fix**    : Garder uniquement AddCors() avec policy "AngularApp"
**Prevention** : Une seule source de verite pour CORS

## HTTP vs HTTPS
**Erreur** : Angular appelle http mais API tourne sur https
**Cause**  : Port 7063 = HTTPS, Angular utilise http://
**Fix**    : Mettre https://localhost:7063 dans environment.ts
**Prevention** : Toujours verifier launchSettings.json pour le bon protocole

## EntityState.Modified sur entite deja trackee
**Erreur**     : `_context.Entry(task).State = EntityState.Modified` corrompt IsDeleted/CreatedAt
**Cause**      : Force toutes les colonnes comme modifiees, y compris les champs d audit
**Fix**        : Guard `if (_context.Entry(entity).State == EntityState.Detached) _context.Update(entity);`
**Prevention** : Ne jamais forcer EntityState.Modified sans verifier le tracking

## Double fetch dans UpdateAsync / SoftDeleteAsync (repository)
**Erreur**     : Repository refaisait un SELECT alors que le service avait deja charge l entite
**Cause**      : Anti-pattern — requete SQL redondante, peut masquer une incoherence d etat
**Fix**        : Accepter l entite en parametre, pas l ID. SoftDeleteAsync(TaskItem task) au lieu de SoftDeleteAsync(int id)
**Prevention** : Les methodes de mutation du repository prennent l entite, pas l ID

## HasQueryFilter sur propriete de navigation (EF Core)
**Erreur**     : `HasQueryFilter(tc => !tc.Task.IsDeleted)` dans une config de table join
**Cause**      : Genere des JOINs implicites, erreur si navigation non chargee
**Fix**        : Supprimer. Le filtrage soft-delete reste au service/repository qui gere l entite parente
**Prevention** : HasQueryFilter uniquement sur proprietes directes de l entite, jamais sur navigations

## HasMaxLength manquant sur colonnes enum en string
**Erreur**     : `HasConversion<string>()` sans `HasMaxLength()` -> nvarchar(max) en base
**Cause**      : EF Core ne deduit pas la longueur d un enum converti en string
**Fix**        : Toujours `HasConversion<string>().HasMaxLength(20)` (ou valeur adaptee)
**Prevention** : Regle systematique : tout HasConversion<string>() suivi de HasMaxLength()

## Entite sans CreatedAt/UpdatedAt + DateTime sans defaut C#
**Erreur**     : `InvitedAt` sans `= DateTime.UtcNow` -> vaut DateTime.MinValue hors EF Core (tests, seed)
**Cause**      : HasDefaultValueSql s applique uniquement en base, pas lors de l instanciation C#
**Fix**        : `public DateTime InvitedAt { get; set; } = DateTime.UtcNow;` dans l entite
**Prevention** : Toute entite avec CreatedAt/UpdatedAt/InvitedAt doit avoir = DateTime.UtcNow en C# ET HasDefaultValueSql en config EF

## Scope imbrique dans DatabaseSeeder
**Erreur**     : SeedAsync recoit un serviceProvider scopé et recrée un scope interne -> ObjectDisposedException
**Cause**      : Le scope parent est cree dans Program.cs, creer un scope enfant et le disposer libere le DbContext trop tot
**Fix**        : Utiliser directement `serviceProvider.GetRequiredService<AppDbContext>()` sans CreateScope()
**Prevention** : Si SeedAsync recoit un IServiceProvider deja scope, ne jamais appeler CreateScope() dessus

## Mot de passe seed hardcode dans le code source
**Erreur**     : "Demo1234" ou "Seed@12345!" en dur dans DatabaseSeeder.cs
**Cause**      : Violation OWASP A02 — attaquant peut s authentifier avec le compte seed en production
**Fix**        : Lire depuis `configuration["SeedData:DefaultPassword"] ?? throw new InvalidOperationException(...)`
**Prevention** : Aucun mot de passe dans le code source, meme pour le seed dev

## GetUserId() avec int.Parse + null-forgiving operator
**Erreur**     : `int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!)` -> FormatException/NullReferenceException non geree
**Cause**      : Si le claim est absent ou invalide, exception echappant au GlobalExceptionMiddleware -> 500 non structure
**Fix**        : `int.TryParse` avec `throw new UnauthorizedException(...)` si parsing echoue
**Prevention** : Toujours valider les claims JWT avant parsing, jamais de ! sur FindFirstValue

## Mutation manuelle entite au lieu d AutoMapper dans UpdateAsync
**Erreur**     : Affectations `taskItem.Title = request.Title` alors qu un profil AutoMapper existait
**Cause**      : Si un champ est ajoute au DTO, le mapper se met a jour mais pas les affectations manuelles -> donnees partielles
**Fix**        : `_mapper.Map(request, taskItem)` pour mettre a jour l entite trackee directement
**Prevention** : Toujours utiliser AutoMapper pour les mappings Update, jamais d affectations manuelles

## Seed sans try/catch dans Program.cs
**Erreur**     : Bloc seed non protege -> crash au demarrage si base inaccessible ou config manquante
**Cause**      : Exception non catchee pendant le demarrage -> app ne demarre pas, message peu clair
**Fix**        : Encapsuler dans try/catch avec `logger.LogError(ex, ...)`, l app continue meme si seed echoue
**Prevention** : Toute operation de demarrage non critique (seed, migration check) doit etre dans un try/catch
