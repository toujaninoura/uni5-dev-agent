# Rules - Security OWASP

## Authentification JWT
- [Authorize] sur tous les endpoints sauf auth
- [AllowAnonymous] uniquement sur register, login, refresh
- Rotation refresh tokens a chaque utilisation
- Expiration : access 60min / refresh 7 jours

## Validation des entrees (OWASP A03)
- FluentValidation sur tous les DTOs entrants
- HasMaxLength() sur tous les strings
- Valider les types : email, url, date
- Rejeter les champs inconnus

## Rate Limiting (OWASP A04)
- 5 tentatives login par minute par IP
- 100 requetes par minute par utilisateur
- Retourner 429 Too Many Requests avec Retry-After

## Headers de securite obligatoires
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000

## Donnees sensibles (OWASP A02)
- Jamais de mot de passe dans les logs ou reponses
- Jamais de stack trace en production
- Jamais de secrets dans le code -> appsettings.json
- .env dans .gitignore

## Injection SQL (OWASP A03)
- EF Core uniquement -> jamais de SQL brut
- Si SQL brut necessaire -> requetes parametrees obligatoires
- Jamais de concatenation de chaines dans les requetes

## Angular - XSS
- Jamais de innerHTML -> utiliser Angular binding
- Jamais de bypassSecurityTrust sauf cas justifie
- Sanitizer sur toutes les entrees utilisateur

## Angular - CSRF
- HttpOnly cookies si sessions
- SameSite=Strict sur les cookies
- CSRF token sur les formulaires sensibles
