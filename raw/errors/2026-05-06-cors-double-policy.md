# Source brute — Erreur CORS double policy
Date    : 2026-05-06
Projet  : task-manager
Erreur  : Provisional headers are shown — Angular ne peut pas appeler l API
Contexte: Login page Angular -> API .NET
Cause   : AddCorsPolicy() ET AddCors() en meme temps dans Program.cs
          app.UseCors("AllowAngular") ET app.UseCors("AngularApp") en double
Fix     : Garder uniquement AddCors() avec policy "AngularApp"
          Une seule app.UseCors("AngularApp") avant UseAuthentication
