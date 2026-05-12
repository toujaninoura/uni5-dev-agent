---
Title: TaskManager Angular — Architecture composants
Updated: 2026-05-11
Sources: dev session 2026-05-11
Raw: ../../raw/architecture/2026-05-11-register-angular-components.md
---

# TaskManager Angular — Architecture composants

## Routes

| Path | Composant | Guard |
|---|---|---|
| /login | LoginComponent | — |
| /register | RegisterComponent | — |
| /tasks | (lazy) TASKS_ROUTES | authGuard |
| ** | redirect /login | — |

## Features — Auth

### LoginComponent
- Path : `features/auth/login/`
- Formulaire Reactive Forms : email, password
- isLoading boolean : guard double-soumission, reset dans next et error
- errorMessage : alerte Bootstrap sur échec
- Lien routerLink="/register" → page inscription
- Redirection vers /tasks après succès

### RegisterComponent
- Path : `features/auth/register/`
- Formulaire Reactive Forms : firstName, lastName, email, password, confirmPassword
- Validator cross-field : passwordMatchValidator (group validator sur le FormGroup)
- isLoading boolean : spinner Bootstrap, bouton disabled, guard double-soumission
- errorMessage : alerte Bootstrap sur erreur API (ex : email déjà utilisé)
- Barre force mot de passe : Bootstrap progress-bar via PasswordStrengthService + ngClass
- Redirection vers /login après succès
- Lien routerLink="/login" → page connexion

## Core Services

### AuthService
- `login(request: AuthRequest)` → POST /api/v1/auth/login
- `register(request: RegisterRequest)` → POST /api/v1/auth/register
- RegisterRequest : `{ firstName, lastName, email, password }`
- `getUserId()` : décode le JWT, vérifie exp avant de retourner l'id

### PasswordStrengthService
- `evaluate(password: string): PasswordStrength`
- Score 0-5 : length>=8, uppercase, lowercase, digit, special char (+1 chacun)
- cssClass : `bg-danger` (≤1) / `bg-warning` (2-3) / `bg-success` (≥4)
- widthPercent : `(score/5)*100%`

## Modèles (core/models/)

| Interface | Champs |
|---|---|
| AuthRequest | email, password |
| RegisterRequest | firstName, lastName, email, password |
| AuthResponse | token, email, userId, expiresAt |

## Conventions tests Angular
- Standalone components : `imports: [Component, ReactiveFormsModule]`
- Router : `provideRouter([]) + provideLocationMocks()` (pas RouterTestingModule)
- Spy sur router.navigate : `router = TestBed.inject(Router); spyOn(router, 'navigate')`
- Services providedIn root : déclarer dans `providers: [ServiceName]` dans TestBed

## See Also
- [Erreurs Angular connues](./errors.md)
- [Architecture backend TaskManager](../dotnet/architecture.md)
