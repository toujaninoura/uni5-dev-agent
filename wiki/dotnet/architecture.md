---
Title: TaskManager — Architecture et entités
Updated: 2026-05-07
Sources: dev session 2026-05-07
Raw: ../../raw/architecture/2026-05-07-task-sharing-feature.md
---

# TaskManager — Architecture et entités

## Entités Domain

### User
- Id (int), Email, PasswordHash, CreatedAt
- Navigation : Tasks, CollaborationsReceived, CollaborationsGiven

### TaskItem
- Id (int), Title, Description, Status, Priority, DueDate, CreatedAt, UpdatedAt, IsDeleted, DeletedAt
- UserId (FK → Users)
- Navigation : Collaborators
- HasQueryFilter : `!t.IsDeleted`

### TaskCollaborator
- Id (int), TaskId (FK → Tasks, Cascade), UserId (FK → Users invité, Restrict), InvitedByUserId (FK → Users invitant, Restrict)
- Role (TaskShareRole : Owner/Editor/Viewer) → HasConversion<string>().HasMaxLength(20)
- Status (InvitationStatus : Pending/Accepted/Rejected) → HasConversion<string>().HasMaxLength(20)
- InvitedAt = DateTime.UtcNow + HasDefaultValueSql("GETUTCDATE()")
- RespondedAt (DateTime?)
- Index unique (TaskId, UserId)

## Enums Domain

| Enum | Valeurs |
|---|---|
| TaskItemStatus | Todo, InProgress, Done |
| TaskPriority | Low, Medium, High |
| TaskShareRole | Owner, Editor, Viewer |
| InvitationStatus | Pending, Accepted, Rejected |

## Endpoints REST

### Auth — /api/v1/auth
| Méthode | Route | Code |
|---|---|---|
| POST | register | 201 |
| POST | login | 200 |

### Tasks — /api/v1/tasks [Authorize]
| Méthode | Route | Description | Code |
|---|---|---|---|
| GET | / | Liste paginée (owner uniquement) | 200 |
| GET | /{id:int} | Détail (owner OU collaborateur accepté) | 200 |
| POST | / | Créer (owner) | 201 |
| PUT | /{id:int} | Modifier (owner OU editor) | 200 |
| DELETE | /{id:int} | Soft-delete (owner uniquement) | 204 |

### Task Sharing — /api/v1/tasks [Authorize]
| Méthode | Route | Description | Code |
|---|---|---|---|
| POST | /{id:int}/collaborators | Inviter (owner uniquement) | 201 |
| DELETE | /{id:int}/collaborators/{userId} | Retirer | 204 |
| GET | /{id:int}/collaborators | Lister (owner OU collaborateur) | 200 |
| GET | /shared | Tâches partagées acceptées | 200 |
| GET | /shared/pending | Invitations en attente | 200 |
| POST | /{id:int}/collaborators/accept | Accepter invitation | 200 |
| POST | /{id:int}/collaborators/reject | Refuser invitation | 200 |

Note : GET /shared ne conflicte pas avec GET /{id:int} grâce à la contrainte `:int`.

## Services Application

| Service | Interface | Responsabilité |
|---|---|---|
| AuthService | IAuthService | Register, Login, génération JWT |
| TaskService | ITaskService | CRUD tâches + contrôle d'accès par rôle |
| TaskSharingService | ITaskSharingService | Invitation, acceptation, rejet, retrait, liste |

## IUnitOfWork

```csharp
ITaskRepository Tasks { get; }
ITaskCollaboratorRepository Collaborators { get; }
Task<int> SaveChangesAsync(CancellationToken ct);
```

## Règles de contrôle d'accès

| Action | Owner | Editor | Viewer | Non-collaborateur |
|---|---|---|---|---|
| GET /tasks | ✅ ses tâches | — | — | ❌ |
| GET /tasks/{id} | ✅ | ✅ | ✅ | ❌ 404 |
| POST /tasks | ✅ | ❌ | ❌ | ❌ |
| PUT /tasks/{id} | ✅ | ✅ | ❌ 401 | ❌ 401 |
| DELETE /tasks/{id} | ✅ | ❌ 401 | ❌ 401 | ❌ 401 |

## Migrations EF Core

| Migration | Description |
|---|---|
| AddUserTable | Table Users |
| AddTaskItemForeignKey | FK UserId sur Tasks |
| UpdateDescriptionMaxLength | MaxLength(1000) sur Description |
| UpdateDeleteBehaviorRestrict | DeleteBehavior.Restrict |
| CompleteTaskItemConfiguration | Index, HasQueryFilter |
| AddTaskCollaboratorTable | Table TaskCollaborators + index unique |

## Conventions importantes

- `GetByTaskAndUserAsync` → AsNoTracking (lecture seule)
- `GetByTaskAndUserTrackingAsync` → avec tracking (pour Update dans Accept/Reject)
- Validation FluentValidation TOUJOURS en premier dans les méthodes de service (fail-fast)
- InviteCollaboratorAsync : validation → owner check → email lookup → self-check → duplicate check → create

## See Also
- [Erreurs .NET connues — EntityState, HasQueryFilter, HasMaxLength, etc.](./errors.md)
