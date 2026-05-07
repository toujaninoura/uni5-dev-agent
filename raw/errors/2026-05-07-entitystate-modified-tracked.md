# Source brute — EntityState.Modified sur entite trackee
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : Corruption silencieuse de IsDeleted/CreatedAt/DeletedAt lors d un UpdateAsync
Contexte: TaskRepository.UpdateAsync apres correction du double fetch — `_context.Entry(task).State = EntityState.Modified` introduit pour remplacer le SELECT redondant
Fix     : Guard `if (_context.Entry(entity).State == EntityState.Detached) _context.Update(entity);` — si entite deja trackee, EF Core detecte les changements automatiquement
