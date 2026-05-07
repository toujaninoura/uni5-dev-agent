# Source brute — HasMaxLength manquant sur colonnes enum en string
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : HasConversion<string>() sans HasMaxLength() -> nvarchar(max) en base (confirme dans migration)
Contexte: TaskCollaboratorConfiguration colonnes Role et Status
Fix     : HasConversion<string>().HasMaxLength(20) — longueur suffisante pour les valeurs d enum
