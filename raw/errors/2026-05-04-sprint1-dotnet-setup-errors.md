---
Source: errors-memory.json (migration depuis ancien systeme)
Collected: 2026-05-12
Published: 2026-05-04
---

# Erreurs rencontrées — Sprint 1 (Setup .NET + Git)

## CS0234 — ILogger non trouvé dans le projet Application
Erreur : CS0234 "Le nom de type ou d espace de noms 'Extensions' n existe pas dans l espace de noms 'Microsoft'" — ILogger introuvable dans le projet Application
Cause : Les projets classlib ne référencent pas Microsoft.Extensions.Logging.Abstractions par défaut. ILogger est utilisé dans Application mais le package n est pas installé.
Fix : dotnet add TaskManager.Application package Microsoft.Extensions.Logging.Abstractions --version 8.0.0
Prevention : Toujours ajouter Microsoft.Extensions.Logging.Abstractions à tout projet classlib qui utilise ILogger

## CS1061 — AddAutoMapper introuvable sur IServiceCollection
Erreur : CS1061 "IServiceCollection does not contain definition for AddAutoMapper"
Cause : Le package AutoMapper seul ne fournit pas l extension DI. Il faut AutoMapper.Extensions.Microsoft.DependencyInjection en complément.
Fix : dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
Prevention : Toujours installer AutoMapper.Extensions.Microsoft.DependencyInjection avec AutoMapper pour le support DI

## GitHub PR impossible — repo vide sans branche main
Erreur : PR GitHub échoue — "No commits between main and feat branch" quand le repo est vide
Cause : Repo GitHub entièrement vide (pas de branche main). La branche feature créée devient orpheline. Après création de main, les branches n ont pas d historique commun.
Fix : Pousser un commit vide sur main en premier, puis créer la branche feature depuis main, puis créer la PR
Prevention : Avant de créer une branche feature, toujours vérifier que main existe sur GitHub avec au moins un commit
