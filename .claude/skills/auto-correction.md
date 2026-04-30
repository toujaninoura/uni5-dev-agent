---
name: auto-correction
description: Lire la memoire des erreurs passees et les eviter automatiquement
---

# Skill - Auto-correction Memory

## PRINCIPE
Avant de commencer le developpement d une issue :
1. Lire errors-memory.json
2. Identifier les erreurs pertinentes pour cette issue
3. Appliquer les corrections proactivement
4. Apres le developpement, sauvegarder les nouvelles erreurs

## LIRE LA MEMOIRE AU DEMARRAGE
Lire errors-memory.json et afficher :
## FORMAT DES ERREURS EN MEMOIRE

### Erreur .NET
{
  "id": "dotnet_{N}",
  "date": "{date}",
  "context": "{description du contexte}",
  "error": "{message d erreur exact}",
  "cause": "{pourquoi ca arrive}",
  "fix": "{comment corriger}",
  "prevention": "{comment eviter la prochaine fois}",
  "tags": ["{tag1}", "{tag2}"]
}

### Erreur Angular
{
  "id": "angular_{N}",
  "date": "{date}",
  "context": "{description du contexte}",
  "error": "{message d erreur exact}",
  "cause": "{pourquoi ca arrive}",
  "fix": "{comment corriger}",
  "prevention": "{comment eviter la prochaine fois}",
  "tags": ["{tag1}", "{tag2}"]
}

## SAUVEGARDER UNE NOUVELLE ERREUR
Apres chaque erreur rencontree et corrigee :
1. Lire errors-memory.json
2. Ajouter l erreur dans la bonne categorie
3. Sauvegarder le fichier

## ERREURS DEJA CONNUES A EVITER

### .NET ? JWT Secret trop court
Prevention : Toujours generer un secret de 32+ caracteres.
Fix auto   : Utiliser "ProjectName_SuperSecretKey_2026!!" comme template.
Jamais     : Mettre un placeholder comme __REPLACE_WITH_SECRET__

### .NET ? Commentaires dans appsettings.json
Prevention : JSON n accepte pas les commentaires //.
Fix auto   : Jamais mettre de commentaires // dans les fichiers .json

### .NET ? Migrations manquantes
Prevention : Toujours creer la migration apres avoir cree le DbContext.
Fix auto   : Verifier dotnet ef migrations list avant dotnet run

### Angular ? Subscribe sans unsubscribe
Prevention : Toujours utiliser async pipe dans les templates.
Fix auto   : Jamais de subscribe() dans ngOnInit sans takeUntilDestroyed

### Angular ? Bootstrap non configure
Prevention : Toujours ajouter Bootstrap dans angular.json styles et scripts.
Fix auto   : Verifier angular.json apres npm install bootstrap
