# Skill — Git Worktree

## Créer
git worktree add ~/projects/{repo}-task-{N} -b feat/issue-{N}-{slug}
cd ~/projects/{repo}-task-{N}

## Supprimer
cd ~/projects/{repo}
git worktree remove ~/projects/{repo}-task-{N} --force
git fetch --prune

## Lister
git worktree list

## Règles
- Max 3 worktrees simultanés
- Un worktree = une issue
- Toujours vérifier la liste avant d'en créer un
- Supprimer immédiatement après merge
