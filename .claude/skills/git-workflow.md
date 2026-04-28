---
name: git-workflow
description: Commandes Git locales - branches, commits, push, rebase, cleanup
---

# Skill - Git Workflow Local

## Initialisation
git init
git add -A
git commit -m "chore: initial commit"
git branch -M main
git remote add origin https://github.com/{owner}/{repo}.git
git push -u origin main

## Nouvelle branche
git checkout main && git pull
git checkout -b feat/issue-{N}-{slug}

## Commit et push
git add -A
git status
git diff --stat
git commit -m "feat(scope): description"
git push origin feat/issue-{N}-{slug}

## Rebase si conflit
git checkout main && git pull
git checkout feat/issue-{N}-{slug}
git rebase main
git rebase --continue
git push --force-with-lease

## Cleanup apres merge
git checkout main && git pull
git branch -d feat/issue-{N}-{slug}
git fetch --prune

## Tags de sprint
git tag sprint-{N}-done
git push origin sprint-{N}-done
