# Rules - Git Workflow

## INTERDIT
- git push --force sur main
- git commit --no-verify
- Committer directement sur main
- Merger sans PR
- Merger sans code review PASSED
- Deux issues dans la meme branche

## OBLIGATOIRE
- Une branche = une issue
- Tout code passe par une PR
- Code review avant chaque merge
- Commits au format conventionnel
- Pull main avant chaque nouvelle branche
- Cleanup branche apres merge

## FORMAT COMMITS STRICT
Types : feat / fix / test / chore / refactor / docs / perf
Format : type(scope): description en minuscules
Exemples :
  feat(api): add products endpoint
  fix(auth): handle null token
  test(users): add register unit tests

## REGLES WORKTREES
- Max 3 worktrees simultanes
- Un worktree = une issue uniquement
- Verifier git worktree list avant d en creer un
- Supprimer immediatement apres merge

## BRANCHES
- Nommage : feat/issue-{N}-{slug}
- Toujours partir de main a jour
- Max 3 branches actives en parallele
- Supprimer apres merge uniquement
