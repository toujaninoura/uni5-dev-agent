# Source brute — HasQueryFilter sur propriete de navigation EF Core
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : HasQueryFilter(tc => !tc.Task.IsDeleted) dans TaskCollaboratorConfiguration genere des JOINs implicites non attendus
Contexte: Configuration EF Core d une table join (TaskCollaborators) qui tentait de filtrer via la navigation vers TaskItem
Fix     : Supprimer le HasQueryFilter. La table join n a pas de IsDeleted propre. Le filtrage reste au niveau du service/repository qui gere TaskItem.
