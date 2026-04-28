---
name: memory-ops
description: Lecture et ecriture de dev-memory.json
---

# Skill - Memory Operations

## Lire la memoire
Get-Content dev-memory.json | ConvertFrom-Json

## Structure dev-memory.json
{
  "project": {
    "name": null,
    "stack": null,
    "repo_url": null,
    "owner": null,
    "repo": null,
    "local_path": null
  },
  "sprint": {
    "current": 1,
    "issues_done": [],
    "issues_pending": []
  },
  "preferences": {
    "commit_style": "conventional",
    "pr_strategy": "squash",
    "branch_naming": "feat/issue-{N}-{slug}",
    "max_parallel_branches": 3
  },
  "instincts": [],
  "decisions": []
}

## Mettre a jour apres chaque issue
Ajouter dans issues_done[] :
{
  "number": {N},
  "title": "{titre}",
  "branch": "feat/issue-{N}-{slug}",
  "pr": {PR_NUMBER},
  "done_at": "{timestamp}"
}

## Sauvegarder les learnings
Ajouter dans instincts[] :
{
  "pattern": "{description du pattern appris}",
  "confidence": 0.9,
  "action": "{ce que l agent doit faire}"
}
