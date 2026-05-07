# Source brute — GetUserId() avec int.Parse + null-forgiving operator
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : int.Parse(User.FindFirstValue(ClaimTypes.NameIdentifier)!) -> FormatException ou NullReferenceException non geree si claim absent/invalide -> reponse 500 non structuree
Contexte: TasksController.GetUserId() — exception echappe au GlobalExceptionMiddleware
Fix     : int.TryParse avec throw new UnauthorizedException("User identity not found in token.") si parsing echoue
