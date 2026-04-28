---
name: dotnet-scaffold
description: Creer un projet .NET N-Tier complet avec toutes les references et packages
---

# Skill - .NET N-Tier Scaffold

## Creer la solution
mkdir C:\projects\{ProjectName}
cd C:\projects\{ProjectName}
dotnet new sln -n {ProjectName}
dotnet new webapi -n {ProjectName}.API
dotnet new classlib -n {ProjectName}.Application
dotnet new classlib -n {ProjectName}.Domain
dotnet new classlib -n {ProjectName}.Infrastructure
dotnet new nunit -n {ProjectName}.Tests

## Ajouter a la solution
dotnet sln add {ProjectName}.API\{ProjectName}.API.csproj
dotnet sln add {ProjectName}.Application\{ProjectName}.Application.csproj
dotnet sln add {ProjectName}.Domain\{ProjectName}.Domain.csproj
dotnet sln add {ProjectName}.Infrastructure\{ProjectName}.Infrastructure.csproj
dotnet sln add {ProjectName}.Tests\{ProjectName}.Tests.csproj

## References entre projets
dotnet add {ProjectName}.API reference {ProjectName}.Application\{ProjectName}.Application.csproj
dotnet add {ProjectName}.API reference {ProjectName}.Infrastructure\{ProjectName}.Infrastructure.csproj
dotnet add {ProjectName}.Application reference {ProjectName}.Domain\{ProjectName}.Domain.csproj
dotnet add {ProjectName}.Infrastructure reference {ProjectName}.Application\{ProjectName}.Application.csproj
dotnet add {ProjectName}.Infrastructure reference {ProjectName}.Domain\{ProjectName}.Domain.csproj
dotnet add {ProjectName}.Tests reference {ProjectName}.Application\{ProjectName}.Application.csproj
dotnet add {ProjectName}.Tests reference {ProjectName}.Domain\{ProjectName}.Domain.csproj

## Packages NuGet

### API
dotnet add {ProjectName}.API package Microsoft.EntityFrameworkCore.Design
dotnet add {ProjectName}.API package Swashbuckle.AspNetCore
dotnet add {ProjectName}.API package FluentValidation.AspNetCore
dotnet add {ProjectName}.API package AutoMapper.Extensions.Microsoft.DependencyInjection
dotnet add {ProjectName}.API package Microsoft.AspNetCore.Authentication.JwtBearer

### Application
dotnet add {ProjectName}.Application package AutoMapper
dotnet add {ProjectName}.Application package FluentValidation
dotnet add {ProjectName}.Application package Microsoft.Extensions.Logging.Abstractions

### Infrastructure
dotnet add {ProjectName}.Infrastructure package Microsoft.EntityFrameworkCore
dotnet add {ProjectName}.Infrastructure package Microsoft.EntityFrameworkCore.SqlServer
dotnet add {ProjectName}.Infrastructure package Microsoft.EntityFrameworkCore.Tools
dotnet add {ProjectName}.Infrastructure package Microsoft.AspNetCore.Identity.EntityFrameworkCore

### Tests
dotnet add {ProjectName}.Tests package NUnit
dotnet add {ProjectName}.Tests package NUnit3TestAdapter
dotnet add {ProjectName}.Tests package Moq
dotnet add {ProjectName}.Tests package FluentAssertions
dotnet add {ProjectName}.Tests package Microsoft.EntityFrameworkCore.InMemory
dotnet add {ProjectName}.Tests package Microsoft.AspNetCore.Mvc.Testing
dotnet add {ProjectName}.Tests package coverlet.collector

## Structure des dossiers
mkdir {ProjectName}.Domain\Entities
mkdir {ProjectName}.Domain\Enums
mkdir {ProjectName}.Domain\Exceptions
mkdir {ProjectName}.Application\Services
mkdir {ProjectName}.Application\Interfaces
mkdir {ProjectName}.Application\DTOs\Requests
mkdir {ProjectName}.Application\DTOs\Responses
mkdir {ProjectName}.Application\Validators
mkdir {ProjectName}.Application\Mappings
mkdir {ProjectName}.Infrastructure\Data\Configurations
mkdir {ProjectName}.Infrastructure\Repositories
mkdir {ProjectName}.API\Controllers
mkdir {ProjectName}.API\Middlewares
mkdir {ProjectName}.API\Extensions
mkdir {ProjectName}.Tests\Unit\Services
mkdir {ProjectName}.Tests\Unit\Validators
mkdir {ProjectName}.Tests\Integration\Controllers

## Verification
dotnet build
dotnet test
