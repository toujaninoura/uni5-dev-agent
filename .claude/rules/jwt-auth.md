# Règles Authentication JWT

## Packages NuGet
dotnet add {Projet}.API package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add {Projet}.API package System.IdentityModel.Tokens.Jwt
dotnet add {Projet}.Infrastructure package Microsoft.AspNetCore.Identity.EntityFrameworkCore

## appsettings.json
"JWT": {
  "Secret": "VotreSecretTresLongAuMoins32Caracteres!",
  "Issuer": "ProductsApi",
  "Audience": "ProductsApiUsers",
  "ExpirationInMinutes": 60,
  "RefreshTokenExpirationInDays": 7
}

## Entités
- User.cs : IdentityUser + FirstName, LastName, CreatedAt, RefreshTokens
- RefreshToken.cs : Id, Token, ExpiresAt, IsRevoked, UserId, User

## Interfaces (Application/Interfaces/)
- IAuthService : RegisterAsync, LoginAsync, RefreshTokenAsync, RevokeTokenAsync
- IJwtService  : GenerateAccessToken, GenerateRefreshToken, GetPrincipalFromExpiredToken

## DTOs
- RegisterRequest  : FirstName, LastName, Email, Password, ConfirmPassword
- LoginRequest     : Email, Password
- RefreshTokenRequest : RefreshToken
- AuthResponse     : AccessToken, RefreshToken, ExpiresAt, UserId, Email, Roles

## Controller (API/Controllers/AuthController.cs)
- POST /api/auth/register
- POST /api/auth/login
- POST /api/auth/refresh
- POST /api/auth/revoke [Authorize]

## Sécuriser les autres controllers
[Authorize] sur tous les controllers sauf AuthController
[AllowAnonymous] uniquement sur les endpoints publics explicites

## Program.cs
- AddIdentity<User, IdentityRole>
- AddAuthentication JwtBearer
- AddSwaggerGen avec SecurityDefinition Bearer
- app.UseAuthentication() AVANT app.UseAuthorization()
