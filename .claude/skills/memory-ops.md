# Skill — Opérations mémoire

## Lire
cat ~/uni5-dev-agent/dev-memory.json

## Mettre à jour un champ
# Utiliser jq ou éditer directement le fichier JSON

## Structure
{
  "project": { name, stack, repo_url, owner, last_sprint },
  "preferences": { commit_style, pr_strategy, branch_naming },
  "answers": { question: réponse },
  "sprint": { current, velocity, dependency_graph },
  "instincts": [{ pattern, confidence, action }],
  "decisions": [{ at, context, decision, outcome, lesson }]
}

## Règles
- Toujours lire avant d'écrire
- Sauvegarder après chaque phase
- Ne jamais effacer les decisions[]
- Incrémenter sprint.current après chaque sprint
