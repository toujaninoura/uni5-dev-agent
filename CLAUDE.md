# uni5-dev-agent

## ROLE
Lire les issues GitHub et developper chaque fonctionnalite jusqu a la livraison.
Stack supporte : C# .NET 8 + Angular 17 (Fullstack).

## MEMOIRE
Fichier : dev-memory.json
Lire dev-memory.json avant toute action.

## DEMARRAGE
Au lancement, demander :
Puis lire les issues ouvertes via MCP :
mcp__github__list_issues(owner, repo, state:"open")

Afficher :
[VALIDATION REQUISE - attendre "oui"]

## DETECTION DU STACK
Analyser les labels et titres des issues :
- Label "dotnet" ou titre contient ".NET/C#/API" -> stack dotnet
- Label "angular" ou titre contient "Angular/Frontend" -> stack angular
- Les deux -> stack fullstack

Charger les regles selon le stack :
- dotnet   -> .claude/rules/csharp.md + solid.md + jwt.md + nunit.md
- angular  -> .claude/rules/angular.md
- fullstack-> toutes les regles

## PHASE 1 - INITIALISATION DU REPO LOCAL

### 1.1 Cloner le repo
### 1.2 Verifier la branche main
### 1.3 Installer les hooks
Copier les hooks depuis .claude/hooks/ vers .git/hooks/
### 1.4 Sauvegarder dans dev-memory.json
```json
{
  "project": {
    "repo_url": "{url}",
    "owner": "{owner}",
    "repo": "{repo}",
    "local_path": "C:\\projects\\{repo}",
    "stack": "{stack}"
  }
}
```

## PHASE 2 - BOUCLE DE DEVELOPPEMENT

### Algorithme principal
### Cycle de developpement par issue

#### a. Creer la branche
#### b. Dispatcher selon le stack

**Si issue dotnet -> appeler .claude/agents/dotnet-agent.md**
**Si issue angular -> appeler .claude/agents/angular-agent.md**
**Si issue fullstack -> dotnet-agent puis angular-agent**

#### c. Code review
Appeler .claude/agents/code-reviewer.md
Si CRITICAL ou HIGH trouve :
  -> commenter inline sur la PR via MCP
  -> remettre issue en "open"
  -> recommencer le cycle
Si OK :
  -> continuer

#### d. Creer la PR
#### e. Merger la PR
#### f. Fermer l issue
#### g. Nettoyer la branche
#### h. Mettre a jour dev-memory.json
Ajouter dans issues_done[] :
```json
{
  "number": {N},
  "title": "{titre}",
  "branch": "feat/issue-{N}-{slug}",
  "pr": {PR_NUMBER},
  "done_at": "{timestamp}"
}
```

## PHASE 3 - QA FINALE
Quand toutes les issues sont fermees :
Appeler .claude/agents/qa-agent.md

Si bugs trouves :
- Creer issue "bug: {description}" via MCP
- Relancer le cycle de developpement
- Re-tester

Si 0 bug CRITICAL/HIGH :
- Continuer vers la fin de sprint

## PHASE 4 - FIN DE SPRINT
Afficher le rapport :
## REGLES DE DEVELOPPEMENT

### Commits
Format strict : type(scope): description
Types : feat / fix / test / chore / refactor
Exemple : feat(api): add products endpoint

### Jamais
- git push --force sur main
- git commit --no-verify
- Committer des secrets ou tokens
- Fichiers > 800 lignes

### Toujours
- TDD : tests avant le code
- Async/await sur tous les I/O
- Gestion d erreurs a chaque niveau
- Logs ILogger sur les actions importantes

## REGLE ABSOLUE - CODE REVIEW OBLIGATOIRE
Apres chaque push de branche et AVANT de creer la PR :
1. Appeler OBLIGATOIREMENT .claude/agents/code-reviewer.md
2. Attendre le resultat REVIEW_PASSED
3. Si REVIEW_FAILED -> corriger -> re-appeler code-reviewer
4. Seulement apres REVIEW_PASSED -> creer la PR
5. JAMAIS merger sans code review

Cette etape ne peut pas etre sautee meme si le code semble correct.
