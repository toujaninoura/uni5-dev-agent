# Source brute — Seed sans try/catch dans Program.cs
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : Bloc SeedAsync non protege -> crash au demarrage si base inaccessible ou config SeedData:DefaultPassword manquante
Contexte: Program.cs appelant DatabaseSeeder.SeedAsync avant app.Run()
Fix     : Encapsuler dans try/catch avec logger.LogError(ex, "An error occurred while seeding the database.") — l app continue de demarrer meme si le seed echoue
