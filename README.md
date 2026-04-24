# uni5-dev-agent

Agent developpeur autonome pour Claude Code.
Lit les issues GitHub et developpe chaque fonctionnalite jusqu a la livraison.

## Ce que fait l agent
1. Lit les issues ouvertes depuis GitHub via MCP
2. Detecte le stack automatiquement
3. Developpe chaque issue dans une branche isolee
4. Cree les PRs et les merge
5. Fait la revue de code avec commentaires inline
6. Valide les criteres d acceptation (QA)
7. Livre le sprint termine

## Stacks supportes
- C# .NET 8 : N-Tier, SOLID, JWT, SQL Server, NUnit
- Angular 17 : Standalone, CSS custom, Jasmine
- Fullstack  : .NET API + Angular frontend

## Installation
git clone https://github.com/toujaninoura/uni5-dev-agent
code uni5-dev-agent

## Utilisation
1. Lancer uni5-BA-agent -> backlog cree sur GitHub
2. Ouvrir uni5-dev-agent dans VS Code
3. Cliquer sur etoile Claude Code
4. Coller : Repo: https://github.com/{owner}/{repo}

## Avec uni5-BA-agent
uni5-BA-agent -> cree les issues
uni5-dev-agent -> developpe les issues

## Licence
MIT - toujaninoura
