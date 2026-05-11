---
Source: internal — dev session 2026-05-07
Collected: 2026-05-07
Published: 2026-05-07
---

# Architecture update — Task Sharing Feature (Sprint 2)

## Nouvelle entité : TaskCollaborator

Table : TaskCollaborators

| Champ | Type | Notes |
|---|---|---|
| Id | int | PK identity |
| TaskId | int | FK → Tasks, DeleteBehavior.Cascade |
| UserId | int | FK → Users (l'invité), DeleteBehavior.Restrict |
| InvitedByUserId | int | FK → Users (l'invitant/owner), DeleteBehavior.Restrict |
| Role | enum TaskShareRole | Converti en string, HasMaxLength(20) |
| Status | enum InvitationStatus | Converti en string, HasMaxLength(20) |
| InvitedAt | DateTime | = DateTime.UtcNow en C# + HasDefaultValueSql("GETUTCDATE()") |
| RespondedAt | DateTime? | Nullable |

Index unique sur (TaskId, UserId).

## Enums

TaskShareRole : Owner, Editor, Viewer
InvitationStatus : Pending, Accepted, Rejected

## Navigations ajoutées

TaskItem.Collaborators : ICollection<TaskCollaborator>
User.CollaborationsReceived : ICollection<TaskCollaborator>
User.CollaborationsGiven : ICollection<TaskCollaborator>

## Nouveaux endpoints (TaskSharingController)

Route base : api/v1/tasks, [Authorize]

| Méthode | Route | Description | Code retour |
|---|---|---|---|
| POST | {id:int}/collaborators | Inviter collaborateur (Owner uniquement) | 201 |
| DELETE | {id:int}/collaborators/{userId} | Retirer collaborateur | 204 |
| GET | {id:int}/collaborators | Lister collaborateurs | 200 |
| GET | shared | Tâches partagées acceptées | 200 |
| GET | shared/pending | Invitations en attente | 200 |
| POST | {id:int}/collaborators/accept | Accepter invitation | 200 |
| POST | {id:int}/collaborators/reject | Refuser invitation | 200 |

Note : GET shared ne conflicte pas avec GET {id:int} grâce à la contrainte :int sur le paramètre.

## Nouveau service : TaskSharingService

Interface : ITaskSharingService (Application/Interfaces/)
Implémentation : TaskSharingService (Application/Services/)

Règles métier :
- InviteCollaboratorAsync : validation FluentValidation EN PREMIER (fail-fast), puis vérification owner, résolution email, check doublon, check self-invite
- AcceptInvitationAsync / RejectInvitationAsync : utiliser GetByTaskAndUserTrackingAsync (avec tracking EF Core) pour éviter InvalidOperationException sur entité détachée
- RemoveCollaboratorAsync : Owner OU self peuvent retirer
- GetCollaboratorsAsync : Owner OU collaborateur accepté peut voir la liste

## Interface ITaskCollaboratorRepository

Méthodes :
- GetByTaskAndUserAsync (AsNoTracking — lecture seule)
- GetByTaskAndUserTrackingAsync (avec tracking — pour Update)
- GetByTaskIdAsync (Include User + InvitedByUser)
- GetSharedWithUserAsync (Status = Accepted, Include Task + InvitedByUser)
- GetPendingForUserAsync (Status = Pending, Include Task + InvitedByUser)
- CreateAsync, UpdateAsync, DeleteAsync
- IsAcceptedCollaboratorAsync, GetUserRoleAsync

## IUnitOfWork étendu

Ajout : ITaskCollaboratorRepository Collaborators

## Modifications TaskService (contrôle d'accès par rôle)

GetByIdAsync : Owner OU collaborateur accepté (IsAcceptedCollaboratorAsync). Retourne 404 dans les deux cas d'échec (ne révèle pas l'existence de la tâche).
UpdateAsync : Owner OU Editor (GetUserRoleAsync). Viewer → UnauthorizedException (401).
DeleteAsync : Owner uniquement (inchangé).
GetAllAsync : Owner uniquement (inchangé).
