# Source brute — Double fetch dans UpdateAsync / SoftDeleteAsync
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : Repository refaisait un SELECT alors que le service avait deja charge l entite. SoftDeleteAsync(int id) retournait silencieusement si null -> SaveChanges sans rien a sauvegarder
Contexte: TaskRepository.UpdateAsync et SoftDeleteAsync
Fix     : Changer signatures : UpdateAsync(TaskItem task) et SoftDeleteAsync(TaskItem task). Le service passe l entite directement, le repository ne refait pas de SELECT.
