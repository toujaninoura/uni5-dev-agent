# Workflow Git obligatoire

## Initialisation du repo
gh repo create {nom-projet} --public --description "{description}"
cd ~/projects/{nom-projet}
git init
git add -A
git commit -m "chore: initial commit"
git branch -M main
git remote add origin https://github.com/toujaninoura/{nom-projet}.git
git push -u origin main

## Protection de la branche main
gh api repos/toujaninoura/{nom-projet}/branches/main/protection \
  --method PUT \
  --field required_pull_request_reviews[required_approving_review_count]=0 \
  --field required_status_checks=null \
  --field enforce_admins=false \
  --field restrictions=null

## Nommage des branches
feat/issue-{N}-{slug}    ??? nouvelle fonctionnalit??
fix/issue-{N}-{slug}     ??? correction de bug
chore/issue-{N}-{slug}   ??? t??che technique
test/issue-{N}-{slug}    ??? ajout de tests

## Cycle complet par t??che (obligatoire)

### 1. Cr??er le worktree isol??
git worktree add ../projects/{nom}-task-{N} -b feat/issue-{N}-{slug}
cd ../projects/{nom}-task-{N}

### 2. D??velopper (dev-agent ??? 8 phases)
# Phase 1 : lire l'issue
# Phase 2 : plan
# Phase 3 : tests NUnit (TDD)
# Phase 4 : impl??mentation
# Phase 5 : dotnet build + dotnet test
# Phase 6 : nettoyage
# Phase 7 : commit conventionnel
# Phase 8 : push

### 3. Commit et push
git add -A
git commit -m "feat(scope): description"
git push origin feat/issue-{N}-{slug}

### 4. Code review
# code-reviewer ??? audit CRITICAL/HIGH/LOW
# fix + commit + push si n??cessaire

### 5. Cr??er la PR
gh pr create \
  --title "feat: {titre de l'issue}" \
  --body "## Changements
{liste des fichiers modifi??s}

## Tests
{r??sum?? des tests ajout??s}

## Checklist
- [ ] dotnet build ??? 0 erreurs
- [ ] dotnet test ??? tous les tests passent
- [ ] Swagger test??

Closes #{N}" \
  --base main \
  --head feat/issue-{N}-{slug}

### 6. Merger la PR
gh pr merge {PR_NUMBER} --squash --delete-branch

# Si conflit :
cd ~/projects/{nom}
git checkout main && git pull
git checkout feat/issue-{N}-{slug}
git rebase main
# R??soudre les conflits
git rebase --continue
git push --force-with-lease
gh pr merge {PR_NUMBER} --squash --delete-branch

### 7. Cleanup
cd ~/projects/{nom}
git worktree remove ../projects/{nom}-task-{N} --force
git fetch --prune
git checkout main && git pull

### 8. Passer ?? la t??che suivante

## Parall??lisme
- T??ches ind??pendantes ??? max 3 worktrees simultan??s
- T??ches d??pendantes ??? attendre le merge avant de commencer
- V??rifier les d??pendances dans le graphe memory.sprint.dependency_graph

## Commits conventionnels
feat(scope): description      ??? nouvelle fonctionnalit??
fix(scope): description       ??? correction bug
test(scope): description      ??? ajout tests
chore(scope): description     ??? t??che technique
refactor(scope): description  ??? refactoring
docs(scope): description      ??? documentation

## Interdit
- git push --force sur main
- git commit --no-verify
- Committer directement sur main
- Merger sans PR

## REGLES WORKTREE
- Max 3 worktrees simultanees
- Un worktree = une issue uniquement
- Toujours verifier git worktree list avant d en creer un nouveau
- Supprimer immediatement apres merge
- Jamais deux issues dans le meme worktree
