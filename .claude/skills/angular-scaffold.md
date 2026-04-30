---
name: angular-scaffold
description: Creer un projet Angular 17 standalone avec CSS custom et structure complete
---

# Skill - Angular 17 Scaffold

## Prerequis
node --version  # v18+
ng version      # Angular CLI installe ?
npm install -g @angular/cli  # si absent

## Creer le projet
cd C:\projects\{ProjectName}
ng new {ProjectName}-frontend --routing --style=css --standalone --strict --skip-git
cd {ProjectName}-frontend

## Configurer les environnements
ng generate environments

## Contenu environment.ts
export const environment = {
  production: false,
  apiUrl: "https://localhost:5001"
};

## Installer ESLint
ng add @angular-eslint/schematics

## Creer la structure
ng generate module core --flat
mkdir src\app\core\services
mkdir src\app\core\guards
mkdir src\app\core\interceptors
mkdir src\app\core\models
mkdir src\app\shared\components
mkdir src\app\shared\directives
mkdir src\app\shared\pipes
mkdir src\app\features\auth
mkdir src\app\features\dashboard

## Generer les fichiers core
ng generate service core/services/auth --skip-tests
ng generate service core/services/api --skip-tests
ng generate interceptor core/interceptors/jwt --skip-tests
ng generate interceptor core/interceptors/error --skip-tests
ng generate guard core/guards/auth --skip-tests

## Configurer le proxy CORS
Creer proxy.conf.json :
{
  "/api": {
    "target": "https://localhost:5001",
    "secure": false,
    "changeOrigin": true
  }
}

## Verification
ng build
ng test --watch=false
ng serve --proxy-config proxy.conf.json

## Installer Bootstrap 5
npm install bootstrap@5
npm install @popperjs/core

## Ajouter dans angular.json -> styles
"styles": [
  "node_modules/bootstrap/dist/css/bootstrap.min.css",
  "src/styles.css"
],
"scripts": [
  "node_modules/bootstrap/dist/js/bootstrap.bundle.min.js"
]
