# Regles C# .NET 8

## Architecture N-Tier obligatoire
{Projet}.Domain/          -> Entites, Exceptions, Enums
{Projet}.Application/     -> Interfaces, Services, DTOs, Validators, Mappings
{Projet}.Infrastructure/  -> DbContext, Repositories, UnitOfWork
{Projet}.API/             -> Controllers, Middlewares, Extensions
{Projet}.Tests/           -> Tests NUnit + Moq

## Conventions
- PascalCase pour classes et methodes
- camelCase pour variables locales
- IMonInterface pour les interfaces
- _maVariable pour les prives
- Async/await obligatoire sur tous les I/O
- ApiResponse<T> sur tous les endpoints
- Pagination sur tous les GET liste
- FluentValidation sur tous les POST/PUT
- AsNoTracking() sur les requetes lecture

## Packages obligatoires
API          : Swashbuckle, FluentValidation.AspNetCore, AutoMapper
Application  : AutoMapper, FluentValidation
Infrastructure : EF Core, SqlServer, Tools
Tests        : NUnit, NUnit3TestAdapter, Moq, FluentAssertions
