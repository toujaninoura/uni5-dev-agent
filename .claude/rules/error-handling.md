# Rules - Error Handling

## .NET - Principe
Une seule facon de gerer les erreurs : GlobalExceptionMiddleware.
Les controllers ne font jamais de try/catch.
Les services lancent des exceptions metier custom.

## Hierarchie des exceptions (Domain/Exceptions/)
AppException (base)
  NotFoundException     -> 404
  ValidationException   -> 400
  BusinessException     -> 422
  UnauthorizedException -> 401
  ForbiddenException    -> 403
  ConflictException     -> 409

## Format AppException
public abstract class AppException : Exception {
  public int StatusCode { get; }
  protected AppException(string message, int statusCode)
    : base(message) => StatusCode = statusCode;
}

public class NotFoundException : AppException {
  public NotFoundException(string name, object key)
    : base($"{name} with key {key} was not found.", 404) {}
}

## GlobalExceptionMiddleware (API/Middlewares/)
- Intercepte toutes les exceptions non gerees
- Retourne toujours ApiResponse<object>.Fail(message)
- Log l exception complete cote serveur
- Jamais exposer la stacktrace en production
- Enregistrer dans Program.cs AVANT app.UseRouting()

## Format reponse erreur uniforme
{
  "success": false,
  "data": null,
  "message": "Product with id 5 was not found.",
  "errors": ["champ1: message", "champ2: message"],
  "timestamp": "2024-01-01T00:00:00Z"
}

## Angular - Principe
Error interceptor global intercepte toutes les erreurs HTTP.
Les composants ne font jamais de try/catch sur les appels API.
Les services propagent les erreurs via throwError().

## Error Interceptor Angular
401 -> rediriger vers /auth/login
403 -> rediriger vers /forbidden
404 -> afficher message "Ressource introuvable"
500 -> afficher message "Erreur serveur, reessayez plus tard"
Autres -> afficher message generique

## Affichage erreurs Angular
- Toujours afficher un message utilisateur comprehensible
- Jamais afficher le message d erreur technique brut
- Utiliser un service de notification global (ToastService)
- Logger les erreurs dans la console en mode dev uniquement
