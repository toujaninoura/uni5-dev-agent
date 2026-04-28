---
name: deployment
description: Build production .NET + Angular - preparation livraison
---

# Skill - Deployment

## .NET - Build production
dotnet publish {Projet}.API -c Release -o ./publish
dotnet publish {Projet}.API -c Release --self-contained true -r win-x64

## .NET - Variables environnement production
appsettings.Production.json :
{
  "ConnectionStrings": {
    "DefaultConnection": "{prod_connection_string}"
  },
  "JWT": {
    "Secret": "{prod_secret_min_32_chars}"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  }
}

## Angular - Build production
ng build --configuration production
-> dist/{ProjectName}-frontend/

## Angular - Variables environnement production
environment.prod.ts :
export const environment = {
  production: true,
  apiUrl: "https://ton-api.com"
};

## Verification avant livraison

### .NET
dotnet build -> 0 erreurs 0 warnings
dotnet test  -> tous les tests passent
dotnet publish -c Release -> build OK

### Angular
ng build --configuration production -> 0 erreurs
ng test --watch=false -> tous les tests passent
ng lint -> 0 warnings

## Checklist livraison sprint
- [ ] dotnet build -> 0 erreurs
- [ ] dotnet test  -> tous les tests passent
- [ ] ng build --configuration production -> 0 erreurs
- [ ] ng test --watch=false -> tous les tests passent
- [ ] Swagger teste : auth + CRUD
- [ ] Angular teste : login + CRUD
- [ ] Variables prod configurees
- [ ] Secrets hors du code
