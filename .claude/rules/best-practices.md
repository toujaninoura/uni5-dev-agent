# Bonnes pratiques obligatoires

## 1. Gestion des erreurs

### Exceptions métier custom (Domain/Exceptions/)
public abstract class AppException : Exception
{
    public int StatusCode { get; }
    protected AppException(string message, int statusCode) : base(message)
        => StatusCode = statusCode;
}
public class NotFoundException : AppException
{
    public NotFoundException(string name, object key)
        : base($"{name} with key {key} was not found.", 404) { }
}
public class BusinessException : AppException
{
    public BusinessException(string message) : base(message, 400) { }
}
public class UnauthorizedException : AppException
{
    public UnauthorizedException(string message) : base(message, 401) { }
}
public class ForbiddenException : AppException
{
    public ForbiddenException(string message) : base(message, 403) { }
}

### Middleware global (API/Middlewares/GlobalExceptionMiddleware.cs)
- Intercepte toutes les exceptions non gérées
- Retourne toujours ApiResponse<object>.Fail(message)
- Log l'exception complète côté serveur
- Ne jamais exposer la stacktrace en production

## 2. Validation

### FluentValidation obligatoire
public class CreateProductRequestValidator : AbstractValidator<CreateProductRequest>
{
    public CreateProductRequestValidator()
    {
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Le nom est obligatoire")
            .MaximumLength(100).WithMessage("Le nom ne peut pas dépasser 100 caractères");

        RuleFor(x => x.Price)
            .GreaterThan(0).WithMessage("Le prix doit être supérieur à 0");

        RuleFor(x => x.Stock)
            .GreaterThanOrEqualTo(0).WithMessage("Le stock ne peut pas être négatif");
    }
}

### Enregistrement dans Program.cs
builder.Services.AddValidatorsFromAssemblyContaining<CreateProductRequestValidator>();
builder.Services.AddFluentValidationAutoValidation();

## 3. Logging

### ILogger partout
public class ProductService : IProductService
{
    private readonly ILogger<ProductService> _logger;

    public async Task<ProductResponse> CreateAsync(CreateProductRequest request)
    {
        _logger.LogInformation("Creating product: {Name}", request.Name);
        try
        {
            var product = await _uow.Products.AddAsync(...);
            _logger.LogInformation("Product created: {Id}", product.Id);
            return _mapper.Map<ProductResponse>(product);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error creating product: {Name}", request.Name);
            throw;
        }
    }
}

## 4. Réponses HTTP standardisées
Toujours utiliser ApiResponse<T> :
- 200 OK      → ApiResponse<T>.Ok(data)
- 201 Created → ApiResponse<T>.Ok(data) avec CreatedAtAction
- 400 Bad Req → ApiResponse<T>.Fail("message")
- 401 Unauth  → ApiResponse<T>.Fail("Unauthorized")
- 404 NotFound→ ApiResponse<T>.Fail("Resource not found")
- 500 Error   → ApiResponse<T>.Fail("Internal server error")

## 5. Pagination
Toujours paginer les listes :
public record PagedRequest(int Page = 1, int PageSize = 10, string? Search = null);
public record PagedResponse<T>(
    IEnumerable<T> Data,
    int Page,
    int PageSize,
    int TotalCount,
    int TotalPages
);

## 6. Sécurité
- Jamais de secrets dans le code → appsettings.json + variables d'environnement
- CORS configuré explicitement (pas de AllowAnyOrigin en production)
- Rate limiting sur les endpoints d'auth
- HTTPS uniquement en production
- Headers de sécurité : X-Content-Type-Options, X-Frame-Options, HSTS

## 7. Performance
- Async/await sur tous les I/O
- AsNoTracking() sur les requêtes en lecture seule
- Select() pour ne charger que les colonnes nécessaires
- Index sur les colonnes fréquemment filtrées
- Pagination obligatoire sur toutes les listes

## 8. Documentation
- Swagger activé avec descriptions XML
- Commentaires XML sur tous les endpoints publics
- README.md à la racine avec instructions de démarrage

## 9. Clean Code
- Méthodes max 20 lignes
- Pas de magic numbers → constantes nommées
- Nommage explicite → pas d'abréviations obscures
- Pas de code commenté → Git garde l'historique
- Un niveau d'abstraction par méthode
