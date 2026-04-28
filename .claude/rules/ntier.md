# Rules - Architecture N-Tier

## Couches obligatoires
{Projet}.Domain          <- couche 1 - entites metier pures
{Projet}.Application     <- couche 2 - logique metier
{Projet}.Infrastructure  <- couche 3 - acces donnees
{Projet}.API             <- couche 4 - presentation
{Projet}.Tests           <- couche 5 - tests

## Dependances autorisees
API            -> Application + Infrastructure
Application    -> Domain uniquement
Infrastructure -> Domain + Application (interfaces)
Domain         -> RIEN (aucune dependance externe)
Tests          -> toutes les couches

## Interdit absolument
API            -> Domain directement (bypasser Application)
Domain         -> Application ou Infrastructure
Infrastructure -> API
Circular dependencies entre couches

## Regles par couche

### Domain
- Entites pures sans attributs EF Core
- Pas de reference a des packages NuGet externes
- Pas de logique metier complexe -> Application
- Uniquement : proprietes, enums, exceptions, value objects

### Application
- Ne connait PAS EF Core
- Depende uniquement des interfaces IRepository
- Contient TOUTE la logique metier
- Un service = une responsabilite (SRP)
- Toujours retourner des DTOs, jamais des entites

### Infrastructure
- Seule couche qui connait EF Core
- Implemente les interfaces definies dans Application
- Pas de logique metier -> uniquement acces donnees
- AppDbContext herite de IdentityDbContext<User> si auth

### API
- Controllers = facades uniquement
- Aucune logique metier dans les controllers
- Appelle uniquement les services via interface
- Retourne uniquement des DTOs dans ApiResponse<T>
- Gestion erreurs via GlobalExceptionMiddleware uniquement

## Injection de dependances (Program.cs)
Repositories -> Scoped
Services     -> Scoped
DbContext    -> Scoped
JWT          -> Singleton
AutoMapper   -> Singleton
