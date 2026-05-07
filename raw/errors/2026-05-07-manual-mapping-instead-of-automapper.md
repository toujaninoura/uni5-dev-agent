# Source brute — Mutation manuelle d entite au lieu d AutoMapper dans UpdateAsync
Date    : 2026-05-07
Projet  : task-manager
Issue   : #27 — TaskCollaborator entity + EF Core migration
Erreur  : taskItem.Title = request.Title, taskItem.Description = request.Description, etc. manuellement alors qu un profil AutoMapper UpdateTaskItemRequest -> TaskItem existait dans TaskItemProfile
Contexte: TaskService.UpdateAsync — si un champ est ajoute au DTO, le mapper se met a jour mais pas les affectations manuelles -> donnees partiellement mises a jour silencieusement
Fix     : _mapper.Map(request, taskItem) — AutoMapper met a jour l entite trackee directement
