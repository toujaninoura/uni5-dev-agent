# Source brute — Mot de passe seed hardcode dans le code source
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : "Demo1234" puis "Seed@12345!" en dur dans DatabaseSeeder.cs — violation OWASP A02
Contexte: Compte demo@taskmanager.com seede avec mot de passe connu de tous ceux qui lisent le code
Fix     : configuration["SeedData:DefaultPassword"] ?? throw new InvalidOperationException("SeedData:DefaultPassword is not configured")
         Valeur dans appsettings.Development.json uniquement
