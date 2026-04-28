# Règles C# / .NET 8

## Stack obligatoire
- Language : C# 12 / .NET 8
- Framework : ASP.NET Core Web API
- ORM : Entity Framework Core
- Tests : NUnit + Moq
- Validation : FluentValidation
- Mapping : AutoMapper

## Commandes
- Build   : dotnet build
- Test    : dotnet test
- Run     : dotnet run
- Format  : dotnet format

## Conventions de nommage
- Classes et méthodes : PascalCase
- Variables locales   : camelCase
- Interfaces          : IMonInterface
- Privés              : _maVariable
- Constants           : MAJUSCULES

## Principes obligatoires
- Async/await sur tous les appels I/O
- Jamais de .Result ou .Wait() (deadlock)
- Nullable reference types activé
- Gestion d'erreurs via middleware global
- Logs via ILogger partout
