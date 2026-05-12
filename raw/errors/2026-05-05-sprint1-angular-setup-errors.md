---
Source: errors-memory.json (migration depuis ancien systeme)
Collected: 2026-05-12
Published: 2026-05-05
---

# Erreurs rencontrées — Sprint 1 (Setup Angular)

## npm install bloqué — UNABLE_TO_VERIFY_LEAF_SIGNATURE
Erreur : npm install bloque pendant 20+ minutes avec UNABLE_TO_VERIFY_LEAF_SIGNATURE et ENOTFOUND, chaque package prenant 21-70 secondes
Cause : npm strict-ssl=true provoque des echecs de verification SSL contre registry.npmjs.org sur cette machine. npm retente 3 fois par package avant de tomber en cache, causant une lenteur extreme.
Fix : npm config set strict-ssl false puis npm install --no-audit --no-fund. Tuer aussi tout process npm install concurrent avant de relancer.
Prevention : Au démarrage d un projet Angular : 1) vérifier npm config get strict-ssl 2) si true, set false 3) jamais lancer plusieurs npm install en parallèle sur le même répertoire 4) utiliser --no-audit --no-fund

## ng new en arrière-plan + npm install concurrent
Erreur : ng new lancé en background crée le répertoire mais les npm install concurrents se bloquent mutuellement — aucun ne se termine
Cause : ng new en background lance npm install en interne. Lancer un npm install séparé en parallèle crée une race condition — les deux se bloquent mutuellement.
Fix : Attendre la fin complète de ng new avant tout npm install supplémentaire. Ou tuer tous les process node.exe concurrents avant un fresh npm install.
Prevention : Après ng new : 1) attendre la fin complète 2) vérifier que node_modules existe 3) seulement ensuite installer les packages supplémentaires comme bootstrap

## ng test --watch=false timeout Vitest worker
Erreur : ng test --watch=false timeout avec "Vitest worker timeout" quand tous les spec files tournent ensemble
Cause : Le pool de workers Vitest timeout quand 5+ spec files tournent simultanément. Chaque worker doit initialiser Angular TestBed, opération coûteuse.
Fix : Ajouter tsConfig dans les options test de angular.json. Lancer les tests par fichier avec --include pendant le développement.
Prevention : Configurer tsConfig dans les options du builder test dans angular.json. Les fichiers de test tournent individuellement pendant le dev.

## ng new génère Angular 21 au lieu d Angular 17 avec Vitest au lieu de Jasmine
Erreur : ng new génère un projet Angular 21 (pas Angular 17) avec Vitest par défaut au lieu de Jasmine+Karma
Cause : Angular CLI 21.x est installé globalement. ng new crée toujours un projet correspondant à la version du CLI. Angular 21 utilise Vitest par défaut.
Fix : Adapter les spec files pour Vitest (vi.fn(), vi.spyOn() au lieu de jasmine.createSpyObj). Utiliser provideRouter([]) au lieu de RouterTestingModule. Utiliser provideHttpClient() + provideHttpClientTesting() au lieu de HttpClientTestingModule.
Prevention : Vérifier ng version avant d écrire les tests. Si Angular 21+ : utiliser les globals Vitest (pas d import nécessaire, configuré dans tsconfig.spec.json types). Utiliser les functional providers, pas les imports de modules deprecated.
