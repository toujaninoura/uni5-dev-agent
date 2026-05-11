# .NET Sprint 2 — Erreurs rencontrées lors du développement de la feature task sharing

> Source: internal — dev session 2026-05-07 (issues #28 #29 #30 #35, code review PR #37)
> Collected: 2026-05-07
> Published: 2026-05-07

## ERROR 1 — AsNoTracking sur entité passée à UpdateAsync → InvalidOperationException

Erreur : InvalidOperationException dans AcceptInvitationAsync et RejectInvitationAsync — "The instance of entity type 'TaskCollaborator' cannot be tracked because another instance with the same key value is already being tracked."
Cause : GetByTaskAndUserAsync utilisait AsNoTracking(), puis l'entité retournée était passée à UpdateAsync qui appelait _context.Update(entity). EF Core ne peut pas tracker une entité déjà chargée en no-tracking si une autre instance trackée existe, ou ne sait pas quels champs ont changé.
Fix : Créer GetByTaskAndUserTrackingAsync (sans AsNoTracking) pour les flows Accept/Reject. Supprimer l'appel explicite UpdateAsync — EF Core suit automatiquement les changements sur une entité trackée, SaveChangesAsync() suffit.
Prévention : Règle claire par méthode : GetByXxxAsync = AsNoTracking (lecture seule), GetByXxxTrackingAsync = avec tracking (pour modification). Ne jamais passer une entité AsNoTracking à une méthode de mise à jour.

## ERROR 2 — FluentValidation appelée après les opérations DB (fail-fast violé)

Erreur : La validation s'exécutait après des appels DB (owner check, email lookup) — si la requête était invalide, des requêtes SQL inutiles étaient exécutées.
Cause : ValidateAsync() placé après les premiers GetByIdAsync / FindByEmailAsync dans InviteCollaboratorAsync.
Fix : Déplacer ValidateAsync() en toute première instruction de la méthode de service, avant tout appel à un repository ou service externe.
Prévention : Pattern fail-fast obligatoire — toujours valider en premier dans les méthodes de service. Si validation échoue → throw ValidationException immédiatement, zéro appel DB.

## ERROR 3 — Migration déjà appliquée — conflit __EFMigrationsHistory

Erreur : `dotnet ef database update` échoue — la table existe déjà en base mais la migration n'est pas dans __EFMigrationsHistory (appliquée depuis une autre branche).
Cause : La migration AddTaskCollaboratorTable avait été appliquée depuis une branche de feature précédente (feat/issue-27). EF Core veut créer la table, mais elle existe déjà.
Fix : Insérer manuellement l'entrée dans __EFMigrationsHistory via sqlcmd :
  INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion]) VALUES ('20260507123059_AddTaskCollaboratorTable', '8.0.0');
  puis relancer dotnet ef database update.
Prévention : Avant de créer une migration, vérifier si la table cible existe déjà (`SELECT * FROM __EFMigrationsHistory`). En cas de conflit entre branches, insérer manuellement plutôt que modifier ou supprimer la migration.

## ERROR 4 — CS8625 : null to non-nullable dans ApiResponse<object>.Ok(null, ...)

Erreur : Warning CS8625 — Cannot convert null literal to non-nullable reference type.
Cause : `ApiResponse<object>.Ok(null, "message")` — le premier paramètre de Ok() est de type T (non-nullable), passer null génère CS8625 avec nullable reference types activé.
Fix : Remplacer par `ApiResponse<string>.Ok(string.Empty, "message")` pour les endpoints qui ne retournent pas de données (Accept, Reject, Remove).
Prévention : Quand un endpoint ne retourne pas de données, utiliser ApiResponse<string> avec string.Empty plutôt que ApiResponse<object> avec null.

## ERROR 5 — Tests NUnit cassés après extension de IUnitOfWork

Erreur : NullReferenceException dans TaskServiceTests après avoir ajouté ITaskCollaboratorRepository Collaborators à IUnitOfWork.
Cause : Les tests existants mockaient IUnitOfWork mais ne configuraient pas la propriété Collaborators. Quand TaskService.GetByIdAsync a commencé à appeler _unitOfWork.Collaborators.IsAcceptedCollaboratorAsync(), la propriété retournait null.
Fix : Ajouter dans SetUp() :
  _collaboratorRepositoryMock = new Mock<ITaskCollaboratorRepository>();
  _unitOfWorkMock.Setup(u => u.Collaborators).Returns(_collaboratorRepositoryMock.Object);
  Configurer IsAcceptedCollaboratorAsync() selon le scénario testé.
Prévention : Chaque fois qu'une propriété est ajoutée à IUnitOfWork, vérifier tous les TestFixtures qui mockent IUnitOfWork et ajouter le setup manquant. Utiliser MockBehavior.Strict pour détecter immédiatement les propriétés non mockées.
