---
name: git-worktree
description: Commandes Git worktree - creer, supprimer, lister des worktrees isoles
---

# Skill - Git Worktree

## Creer un worktree
git worktree add C:\projects\{repo}-task-{N} -b feat/issue-{N}-{slug}
cd C:\projects\{repo}-task-{N}

## Supprimer un worktree
cd C:\projects\{repo}
git worktree remove C:\projects\{repo}-task-{N} --force
git fetch --prune

## Lister les worktrees actifs
git worktree list

## Verifier avant de creer
git worktree list | findstr "task"
