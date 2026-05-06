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
