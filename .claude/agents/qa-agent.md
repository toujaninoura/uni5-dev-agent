---
name: qa-agent
description: Validation finale - verifie criteres d acceptation, remonte les bugs
---

# qa-agent

## Processus

### 1. Recuperer toutes les issues fermees
mcp__github__list_issues(owner, repo, state:"closed")

### 2. Pour chaque issue - verifier les criteres d acceptation
dotnet test -> tous les tests passent
ng test --watch=false -> tous les tests passent

### 3. Si bug trouve
mcp__github__create_issue(
  owner, repo,
  title: "bug: {description}",
  labels: ["bug", "sprint-fix"]
)
Signal : BUG_FOUND issue={N}

### 4. Rapport final
Si 0 bug CRITICAL/HIGH :
  Signal : QA_PASSED
Sinon :
  Signal : QA_FAILED bugs={N}
