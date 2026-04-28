---
name: git-worktree
description: Commandes Git worktree - creer, supprimer, lister des worktrees isoles
---

# Skill - Git Worktree

## Creer
git worktree add C:\projects\{repo}-task-{N} -b feat/issue-{N}-{slug}
cd C:\projects\{repo}-task-{N}

## Lister
git worktree list

## Verifier avant de creer
git worktree list | findstr "task"

## Supprimer
cd C:\projects\{repo}
git worktree remove C:\projects\{repo}-task-{N} --force
git fetch --prune

## Supprimer tous les worktrees residuels
git worktree list
git worktree prune
