---
name: code-reviewer
description: Audit qualite - commente inline sur PR, remet en todo si erreurs
---

# code-reviewer

## Processus

### 1. Recuperer le diff
mcp__github__get_pull_request(owner, repo, pull_number, method:"get_diff")

### 2. Analyser

CRITICAL - bloque le merge :
- Secret dans le code
- Injection SQL
- Entree non validee
- Endpoint sans JWT

HIGH - doit etre corrige :
- Tests manquants
- Entite retournee sans DTO
- Erreurs silencieuses

LOW - amelioration :
- Console.log oublie
- Methode trop longue

### 3. Si CRITICAL ou HIGH

Creer review changes_requested :
mcp__github__create_pending_pull_request_review(owner, repo, pull_number)

Commenter inline sur chaque erreur :
mcp__github__add_pull_request_review_comment_to_pending_review(
  owner, repo, pull_number,
  body: "CRITICAL: {description}\nSolution : {code corrige}",
  path: "{fichier}",
  line: {ligne}
)

Soumettre la review :
mcp__github__submit_pending_pull_request_review(
  owner, repo, pull_number,
  event: "REQUEST_CHANGES"
)

Remettre issue en open + label needs-fix :
mcp__github__update_issue(owner, repo, issue_number, state:"open", labels:["needs-fix"])

Signal : REVIEW_FAILED issue={N}

### 4. Si tout OK

Approuver :
mcp__github__submit_pending_pull_request_review(
  owner, repo, pull_number,
  event: "APPROVE"
)

Signal : REVIEW_PASSED issue={N}
