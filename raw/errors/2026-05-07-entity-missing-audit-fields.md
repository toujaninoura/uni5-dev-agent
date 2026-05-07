# Source brute — Entite sans CreatedAt/UpdatedAt + DateTime sans defaut C#
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : TaskCollaborator sans CreatedAt/UpdatedAt. InvitedAt sans valeur par defaut C# -> DateTime.MinValue hors EF Core (tests, seed)
Contexte: HasDefaultValueSql("GETUTCDATE()") s applique uniquement en base, pas lors de l instanciation C#
Fix     : public DateTime InvitedAt { get; set; } = DateTime.UtcNow; dans l entite. Idem pour CreatedAt et UpdatedAt.
