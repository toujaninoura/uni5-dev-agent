# Rules - C# .NET 8

## Stack obligatoire
- Language   : C# 12 / .NET 8
- Framework  : ASP.NET Core Web API
- ORM        : Entity Framework Core
- Tests      : NUnit + Moq
- Validation : FluentValidation
- Mapping    : AutoMapper

## Architecture N-Tier obligatoire
{Projet}.Domain/
  Entities/        <- entites pures sans dependances
  Enums/           <- enumerations metier
  Exceptions/      <- exceptions custom

{Projet}.Application/
  Services/        <- logique metier
  Interfaces/      <- IService, IRepository
  DTOs/            <- Request + Response
  Validators/      <- FluentValidation
  Mappings/        <- profils AutoMapper

{Projet}.Infrastructure/
  Data/            <- AppDbContext + Configurations
  Repositories/    <- implementation IRepository
  Migrations/      <- migrations EF Core

{Projet}.API/
  Controllers/     <- endpoints REST uniquement
  Middlewares/     <- gestion erreurs globale
  Extensions/      <- configuration DI

{Projet}.Tests/
  Unit/            <- tests unitaires services
  Integration/     <- tests controllers

## Conventions de nommage
- Classes et methodes : PascalCase
- Variables locales   : camelCase
- Interfaces          : IMonInterface
- Prives              : _maVariable
- Constants           : MAJUSCULES

## Regles obligatoires
- Async/await sur tous les I/O
- Jamais de .Result ou .Wait()
- Nullable reference types active
- ApiResponse<T> sur tous les endpoints
- Jamais retourner une entite directement -> toujours un DTO
- AsNoTracking() sur les requetes lecture
- Pagination sur tous les GET liste

## Commandes
- Build  : dotnet build
- Test   : dotnet test
- Run    : dotnet run --project {Projet}.API
- Format : dotnet format
