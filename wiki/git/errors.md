---
Title: Git & GitHub — Erreurs connues et solutions
Updated: 2026-05-11
Sources: dev session 2026-05-11
Raw: ../../raw/errors/2026-05-11-sprint3-angular-git-errors.md
---

# Git & GitHub — Erreurs connues et solutions

## PR GitHub non mergeable — branche orpheline
**Erreur**     : `gh pr merge` echoue avec "merge commit cannot be cleanly created" — GitHub suggere de merger `origin/feat/issue-1-setup-ntier`
**Cause**      : Vieille branche feature (ex : `feat/issue-1-setup-ntier`) jamais supprimee sur GitHub. GitHub calcule un ancetre commun errone et detecte un conflit non resolu entre la branche et main
**Fix**        : Merger la feature branch directement en local (`git merge feat/xxx --no-ff`) puis `git push origin main`. Fermer la PR GitHub manuellement avec un commentaire expliquant le merge local
**Prevention** : Supprimer les branches remote immediatement apres chaque merge. Jamais laisser de branches orphelines sur GitHub : `git push origin --delete nom-branche`

## GitHub PR impossible — repo vide sans branche main
**Erreur**     : PR GitHub échoue — "No commits between main and feat branch" au premier PR d un repo neuf
**Cause**      : Repo GitHub entièrement vide (pas de branche main). La branche feature créée devient orpheline. Après création de main, les deux branches n ont pas d historique commun.
**Fix**        : Pousser un commit init sur main en premier (`git commit --allow-empty -m "chore: init"`), puis créer la branche feature depuis main, puis créer la PR
**Prevention** : Avant de créer une branche feature sur un nouveau repo, toujours vérifier que main existe sur GitHub avec au moins un commit

## See Also
- [Erreurs Angular connues](../angular/errors.md)
- [Erreurs .NET connues](../dotnet/errors.md)
