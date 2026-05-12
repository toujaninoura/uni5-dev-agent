---
Source: dev session 2026-05-11
Collected: 2026-05-11
Published: 2026-05-11
---

# Angular Register Components — Sprint 3

## RegisterComponent
- Path : task-manager-ui/src/app/features/auth/register/
- Fichiers : register.component.ts, .html, .css, .spec.ts
- Route : /register (non protégée, sans authGuard)
- Formulaire Reactive Forms : firstName, lastName, email, password, confirmPassword
- Validator cross-field : passwordMatchValidator (group validator)
- isLoading boolean : guard double-soumission, spinner Bootstrap, bouton disabled
- errorMessage : affichage alerte Bootstrap sur erreur API
- Redirection vers /login après succès
- Lien routerLink vers /login

## RegisterRequest (user.model.ts)
- Interface ajoutée : { firstName: string; lastName: string; email: string; password: string; }
- AuthService.register() accepte désormais RegisterRequest (remplace AuthRequest)

## PasswordStrengthService
- Path : task-manager-ui/src/app/core/services/password-strength.service.ts
- providedIn: 'root'
- Méthode : evaluate(password: string): PasswordStrength
- Interface PasswordStrength : { score: number; cssClass: string; widthPercent: string; }
- Score 0-5 : +1 si length>=8, +1 si uppercase, +1 si lowercase, +1 si digit, +1 si special char
- cssClass : score<=1 -> bg-danger, score<=3 -> bg-warning, score>=4 -> bg-success
- widthPercent : (score/5)*100 + '%'

## LoginComponent — Modifications
- Toggle register mode supprimé (LoginComponent = login uniquement)
- isLoading boolean ajouté (guard double-soumission)
- Lien routerLink="/register" ajouté dans le template
- RouterLink importé dans imports standalone

## app.routes.ts
- Route /register ajoutée : { path: 'register', component: RegisterComponent }
- Route sans canActivate (page publique)

## Tests
- RegisterComponent : 18 tests Jasmine (provideRouter + provideLocationMocks)
- PasswordStrengthService : 11 tests Jasmine
- LoginComponent : 8 tests Jasmine
- Total Angular : 125 tests
