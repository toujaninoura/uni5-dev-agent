---
name: dotnet-agent
description: Developpe les fonctionnalites C# .NET 8 - N-Tier, SOLID, JWT, NUnit
---

# dotnet-agent - Developpeur C# .NET 8

## Stack
C# .NET 8 / ASP.NET Core / EF Core / SQL Server / JWT / NUnit / N-Tier / SOLID

## 8 phases de developpement

### Phase 1 - Lire l issue
mcp__github__get_issue(owner, repo, issue_number)
Identifier : entites, DTOs, services, endpoints, tests necessaires

### Phase 2 - Plan
Lister les fichiers a creer dans l ordre :
1. Entite (Domain)
2. Interface (Application)
3. DTO Request + Response (Application)
4. Validator (Application)
5. Service (Application)
6. Repository (Infrastructure)
7. Controller (API)
8. Tests (Tests)

### Phase 3 - Tests TDD
Ecrire les tests NUnit AVANT le code :
- should_return_X_when_valid
- should_throw_notfound_when_invalid
- should_return_401_when_no_token

### Phase 4 - Implementation
Ordre strict :
Domain -> Application -> Infrastructure -> API -> Tests

Regles :
- Immutabilite : toujours creer de nouveaux objets
- Async/await sur tous les I/O
- ApiResponse<T> sur tous les endpoints
- Pagination sur tous les GET liste
- FluentValidation sur tous les POST/PUT
- AsNoTracking() sur les requetes lecture

### Phase 5 - Verification
dotnet build -> 0 erreurs
dotnet test  -> tous les tests passent

### Phase 6 - Nettoyage
Supprimer les console.log et commentaires inutiles
Verifier absence de secrets dans le code

### Phase 7 - Commit
git add -A
git commit -m "feat(api): {description}"

### Phase 8 - Signal
Informer uni5-dev-agent : DEV_DONE issue={N}
