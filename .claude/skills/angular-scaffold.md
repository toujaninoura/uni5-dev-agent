# Skill — Créer un projet Angular complet

## Prérequis
```bash
node --version   # v18+
npm --version    # v9+
ng version       # Angular CLI installé ?

# Installer Angular CLI si absent
npm install -g @angular/cli
```

## Étape 1 — Créer le projet
```bash
cd ~/projects/{ProjectName}
ng new {ProjectName}-frontend \
  --routing \
  --style=css \
  --standalone \
  --strict \
  --skip-git
cd {ProjectName}-frontend
```

## Étape 2 — Configurer l'environnement
```bash
# Générer les fichiers d'environnement
ng generate environments

# Modifier src/environments/environment.ts
cat > src/environments/environment.ts << 'ENV'
export const environment = {
  production: false,
  apiUrl: 'https://localhost:5001'
};
ENV
```

## Étape 3 — Installer ESLint
```bash
ng add @angular-eslint/schematics
```

## Étape 4 — Créer la structure
```bash
# Core
ng generate module core --flat
mkdir -p src/app/core/{services,guards,interceptors,models}

# Shared
ng generate module shared --flat
mkdir -p src/app/shared/{components,directives,pipes}

# Features
mkdir -p src/app/features/{auth,dashboard}

# Features auth
ng generate component features/auth/login --standalone --skip-tests
ng generate component features/auth/register --standalone --skip-tests

# Services core
ng generate service core/services/auth --skip-tests
ng generate service core/services/api --skip-tests

# Intercepteurs
ng generate interceptor core/interceptors/jwt --skip-tests
ng generate interceptor core/interceptors/error --skip-tests

# Guards
ng generate guard core/guards/auth --skip-tests
```

## Étape 5 — Configurer le proxy CORS (dev)
```bash
cat > proxy.conf.json << 'PROXY'
{
  "/api": {
    "target": "https://localhost:5001",
    "secure": false,
    "changeOrigin": true
  }
}
PROXY

# Modifier angular.json → serve → options
# "proxyConfig": "proxy.conf.json"
```

## Étape 6 — Packages à installer
```bash
# Aucune UI library (CSS custom)
# Optionnel selon le besoin :
npm install @auth0/angular-jwt        # gestion JWT
npm install --save-dev karma-coverage # couverture de tests
```

## Étape 7 — Vérification
```bash
ng build          # 0 erreurs
ng test --watch=false  # tous les tests passent
ng lint           # 0 warnings
ng serve          # app accessible sur http://localhost:4200
```

## Commandes utiles
```bash
ng serve                    # démarrer en dev
ng serve --proxy-config proxy.conf.json  # avec proxy API
ng build --configuration production      # build prod
ng test --watch=false --code-coverage    # tests + couverture
ng generate component features/{name}/{component} --standalone
ng generate service core/services/{name}
```
