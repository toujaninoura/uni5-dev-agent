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

## WORKFLOW FULLSTACK - 2 AGENTS DEV

### Ordre de developpement obligatoire
1. dotnet-agent  -> developpe l API .NET en premier
2. code-reviewer -> review la PR .NET
   - REVIEW_FAILED -> dotnet-agent corrige -> code-reviewer re-verifie
   - REVIEW_PASSED -> merger la PR .NET
3. angular-agent -> developpe le frontend APRES merge .NET
4. code-reviewer -> review la PR Angular
   - REVIEW_FAILED -> angular-agent corrige -> code-reviewer re-verifie
   - REVIEW_PASSED -> merger la PR Angular

### Pourquoi cet ordre ?
Angular depend de l API .NET.
L API doit etre mergee sur main avant de commencer Angular.

### Identification de l agent a appeler
Labels de l issue :
- Label "dotnet" ou "api" -> appeler dotnet-agent
- Label "angular" ou "frontend" -> appeler angular-agent
- Pas de label -> analyser le titre :
  - contient "controller/endpoint/API/.NET" -> dotnet-agent
  - contient "component/page/Angular/UI" -> angular-agent

### Code review par agent
Apres dotnet-agent :
- code-reviewer verifie : N-Tier, SOLID, JWT, DTOs, NUnit

Apres angular-agent :
- code-reviewer verifie : Standalone components, async pipe, JWT interceptor, Jasmine

### Signal REVIEW_FAILED
Si dotnet-agent recoit REVIEW_FAILED :
- Lire commentaires PR
- Corriger le code .NET
- Repousser sur la meme branche
- Signal -> code-reviewer re-verifie

Si angular-agent recoit REVIEW_FAILED :
- Lire commentaires PR
- Corriger le code Angular
- Repousser sur la meme branche
- Signal -> code-reviewer re-verifie

## RULES A CHARGER AU DEMARRAGE
- .claude/rules/common.md
- .claude/rules/best-practices.md
- .claude/rules/csharp.md
- .claude/rules/angular.md
- .claude/rules/solid.md
- .claude/rules/database.md
- .claude/rules/jwt-auth.md
- .claude/rules/tdd-nunit.md
- .claude/rules/git-workflow.md
- .claude/rules/error-handling.md
- .claude/rules/api-standards.md
- .claude/rules/security.md
- .claude/rules/performance.md

## SKILLS A CONSULTER SELON LA TACHE
Scaffold .NET    -> .claude/skills/dotnet-scaffold.md
Scaffold Angular -> .claude/skills/angular-scaffold.md
Connexion API    -> .claude/skills/angular-api.md
GitHub ops       -> .claude/skills/github-ops.md
Git local        -> .claude/skills/git-workflow.md
Git worktrees    -> .claude/skills/git-worktree.md
Migrations EF    -> .claude/skills/dotnet-migrations.md
Tests .NET       -> .claude/skills/dotnet-testing.md
Tests Angular    -> .claude/skills/angular-testing.md
Deployment       -> .claude/skills/deployment.md
Memory           -> .claude/skills/memory-ops.md

## DESIGN PATTERNS
Rules  -> .claude/rules/design-patterns.md
Skills -> .claude/skills/design-patterns.md

Charger ces deux fichiers pour chaque issue de developpement.

## MEMOIRE AUTO-CORRECTION ? OBLIGATOIRE

### Au demarrage de chaque sprint
Lire errors-memory.json et afficher :
### Apres chaque sprint
Consolider les nouvelles erreurs dans errors-memory.json.
Identifier les patterns recurrents.
Ajouter dans la section "prevention" des regles permanentes.

### Erreurs critiques deja apprises
1. JWT Secret -> toujours 32+ caracteres, jamais placeholder
2. appsettings.json -> jamais de commentaires //
3. Bootstrap -> toujours dans angular.json styles ET scripts
4. Migrations -> toujours creer et appliquer avant dotnet run
5. CORS -> toujours configurer pour http://localhost:4200

## WIKI DE CONNAISSANCES ? OBLIGATOIRE

### Au demarrage de chaque issue
1. Lire wiki\index.md
2. Lire wiki\dotnet\errors.md (si issue .NET)
3. Lire wiki\angular\errors.md (si issue Angular)
4. Appliquer proactivement toutes les corrections connues

Afficher :
### Apres chaque nouvelle erreur rencontree
1. Ajouter dans wiki\{stack}\errors.md
2. Mettre a jour wiki\log.md avec la date
3. Mettre a jour wiki\index.md stats

Format :
## {Titre erreur}
**Erreur**     : {message exact}
**Cause**      : {pourquoi}
**Fix**        : {solution}
**Prevention** : {comment eviter}

### Requetes wiki disponibles
- "Qu est-ce que je sais sur JWT ?" -> chercher dans wiki/
- "Ingere cet article : {url}"      -> ajouter dans raw/ et wiki/
- "Lint mon wiki"                   -> verifier liens et coherence
