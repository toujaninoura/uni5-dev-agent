---
name: dotnet-migrations
description: Commandes EF Core migrations - creer, appliquer, annuler
---

# Skill - EF Core Migrations

## Creer une migration
dotnet ef migrations add {Name} --project {Projet}.Infrastructure --startup-project {Projet}.API

## Appliquer la migration
dotnet ef database update --project {Projet}.Infrastructure --startup-project {Projet}.API

## Annuler la derniere migration
dotnet ef migrations remove --project {Projet}.Infrastructure --startup-project {Projet}.API

## Revenir a une migration precedente
dotnet ef database update {MigrationName} --project {Projet}.Infrastructure --startup-project {Projet}.API

## Generer le script SQL
dotnet ef migrations script --project {Projet}.Infrastructure --startup-project {Projet}.API

## Verifier les migrations en attente
dotnet ef migrations list --project {Projet}.Infrastructure --startup-project {Projet}.API

## Nommage des migrations
Add{Entity}Table          <- nouvelle entite
Add{Field}To{Entity}      <- nouveau champ
Update{Field}In{Entity}   <- modifier un champ
Add{Entity}Index          <- nouvel index
Remove{Field}From{Entity} <- supprimer un champ

## Installer dotnet-ef si absent
dotnet tool install --global dotnet-ef
dotnet tool update --global dotnet-ef
