# Rules - JWT Authentication

## Packages
dotnet add {Projet}.API package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add {Projet}.Infrastructure package Microsoft.AspNetCore.Identity.EntityFrameworkCore

## appsettings.json
"JWT": {
  "Secret": "VotreSecretTresLongAuMoins32Caracteres!",
  "Issuer": "ProjectApi",
  "Audience": "ProjectApiUsers",
  "ExpirationInMinutes": 60,
  "RefreshTokenExpirationInDays": 7
}

## Entites obligatoires
- User : IdentityUser + FirstName, LastName, CreatedAt, RefreshTokens
- RefreshToken : Id, Token, ExpiresAt, IsRevoked, UserId, User

## Endpoints obligatoires
- POST /api/v1/auth/register
- POST /api/v1/auth/login
- POST /api/v1/auth/refresh
- POST /api/v1/auth/revoke [Authorize]

## Regles securite
- [Authorize] sur tous les endpoints sauf auth
- [AllowAnonymous] uniquement sur register, login, refresh
- Rotation des refresh tokens a chaque utilisation
- Access token : 60 min / Refresh token : 7 jours
- Jamais stocker le mot de passe en clair
- Toujours hasher avec Identity

## Program.cs
- AddIdentity<User, IdentityRole>
- AddAuthentication JwtBearer
- AddSwaggerGen avec SecurityDefinition Bearer
- app.UseAuthentication() AVANT app.UseAuthorization()
