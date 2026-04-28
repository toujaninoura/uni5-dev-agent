---
name: design-patterns
description: Code complet de tous les design patterns - Repository, UoW, Service, DTO, Strategy, Factory, Mediator, Observer, Decorator
---

# Skill - Design Patterns Code

## 1. Repository Pattern

### Interface generique
// Application/Interfaces/IRepository.cs
public interface IRepository<T> where T : class {
  Task<T?> GetByIdAsync(int id);
  Task<IEnumerable<T>> GetAllAsync();
  Task<T> AddAsync(T entity);
  Task UpdateAsync(T entity);
  Task DeleteAsync(int id);
  Task<bool> ExistsAsync(int id);
}

### Interface specifique
// Application/Interfaces/I{Entity}Repository.cs
public interface I{Entity}Repository : IRepository<{Entity}> {
  Task<IEnumerable<{Entity}>> SearchAsync(string query, int page, int size);
  Task<{Entity}?> GetByEmailAsync(string email);
}

### Implementation
// Infrastructure/Repositories/{Entity}Repository.cs
public class {Entity}Repository : I{Entity}Repository {
  private readonly AppDbContext _context;
  public {Entity}Repository(AppDbContext context) => _context = context;

  public async Task<{Entity}?> GetByIdAsync(int id)
    => await _context.{Entities}.AsNoTracking()
        .FirstOrDefaultAsync(e => e.Id == id);

  public async Task<{Entity}> AddAsync({Entity} entity) {
    await _context.{Entities}.AddAsync(entity);
    return entity;
  }

  public async Task UpdateAsync({Entity} entity)
    => _context.{Entities}.Update(entity);

  public async Task DeleteAsync(int id) {
    var entity = await _context.{Entities}.FindAsync(id);
    if (entity != null) { entity.IsDeleted = true; entity.DeletedAt = DateTime.UtcNow; }
  }

  public async Task<bool> ExistsAsync(int id)
    => await _context.{Entities}.AnyAsync(e => e.Id == id);
}

---

## 2. Unit of Work Pattern

### Interface
// Application/Interfaces/IUnitOfWork.cs
public interface IUnitOfWork : IDisposable {
  I{Entity1}Repository {Entities1} { get; }
  I{Entity2}Repository {Entities2} { get; }
  Task<int> SaveChangesAsync();
  Task BeginTransactionAsync();
  Task CommitTransactionAsync();
  Task RollbackTransactionAsync();
}

### Implementation
// Infrastructure/Repositories/UnitOfWork.cs
public class UnitOfWork : IUnitOfWork {
  private readonly AppDbContext _context;
  private IDbContextTransaction? _transaction;

  public I{Entity1}Repository {Entities1} { get; }
  public I{Entity2}Repository {Entities2} { get; }

  public UnitOfWork(AppDbContext context,
    I{Entity1}Repository {entities1},
    I{Entity2}Repository {entities2}) {
    _context = context;
    {Entities1} = {entities1};
    {Entities2} = {entities2};
  }

  public async Task<int> SaveChangesAsync() {
    UpdateTimestamps();
    return await _context.SaveChangesAsync();
  }

  public async Task BeginTransactionAsync()
    => _transaction = await _context.Database.BeginTransactionAsync();

  public async Task CommitTransactionAsync() {
    await _context.SaveChangesAsync();
    await _transaction!.CommitAsync();
  }

  public async Task RollbackTransactionAsync()
    => await _transaction!.RollbackAsync();

  private void UpdateTimestamps() {
    var entries = _context.ChangeTracker.Entries<BaseEntity>()
      .Where(e => e.State == EntityState.Modified);
    foreach (var entry in entries)
      entry.Entity.UpdatedAt = DateTime.UtcNow;
  }

  public void Dispose() => _context.Dispose();
}

---

## 3. Service Pattern

### Interface
// Application/Interfaces/I{Entity}Service.cs
public interface I{Entity}Service {
  Task<{Entity}Response> GetByIdAsync(int id);
  Task<PagedResponse<{Entity}Response>> GetAllAsync(int page, int size, string? search);
  Task<{Entity}Response> CreateAsync(Create{Entity}Request request);
  Task<{Entity}Response> UpdateAsync(int id, Update{Entity}Request request);
  Task DeleteAsync(int id);
}

### Implementation
// Application/Services/{Entity}Service.cs
public class {Entity}Service : I{Entity}Service {
  private readonly IUnitOfWork _uow;
  private readonly IMapper _mapper;
  private readonly ILogger<{Entity}Service> _logger;

  public {Entity}Service(IUnitOfWork uow, IMapper mapper,
    ILogger<{Entity}Service> logger) {
    _uow = uow; _mapper = mapper; _logger = logger;
  }

  public async Task<{Entity}Response> GetByIdAsync(int id) {
    _logger.LogInformation("Getting {Entity} {Id}", nameof({Entity}), id);
    var entity = await _uow.{Entities}.GetByIdAsync(id)
      ?? throw new NotFoundException(nameof({Entity}), id);
    return _mapper.Map<{Entity}Response>(entity);
  }

  public async Task<PagedResponse<{Entity}Response>> GetAllAsync(
    int page, int size, string? search) {
    var items = string.IsNullOrEmpty(search)
      ? await _uow.{Entities}.GetAllAsync()
      : await _uow.{Entities}.SearchAsync(search, page, size);
    var total = items.Count();
    return new PagedResponse<{Entity}Response>(
      _mapper.Map<IEnumerable<{Entity}Response>>(items.Skip((page-1)*size).Take(size)),
      page, size, total, (int)Math.Ceiling(total / (double)size),
      page * size < total, page > 1
    );
  }

  public async Task<{Entity}Response> CreateAsync(Create{Entity}Request request) {
    _logger.LogInformation("Creating {Entity}", nameof({Entity}));
    var entity = _mapper.Map<{Entity}>(request);
    await _uow.{Entities}.AddAsync(entity);
    await _uow.SaveChangesAsync();
    return _mapper.Map<{Entity}Response>(entity);
  }

  public async Task<{Entity}Response> UpdateAsync(int id, Update{Entity}Request request) {
    var entity = await _uow.{Entities}.GetByIdAsync(id)
      ?? throw new NotFoundException(nameof({Entity}), id);
    _mapper.Map(request, entity);
    await _uow.{Entities}.UpdateAsync(entity);
    await _uow.SaveChangesAsync();
    return _mapper.Map<{Entity}Response>(entity);
  }

  public async Task DeleteAsync(int id) {
    var exists = await _uow.{Entities}.ExistsAsync(id);
    if (!exists) throw new NotFoundException(nameof({Entity}), id);
    await _uow.{Entities}.DeleteAsync(id);
    await _uow.SaveChangesAsync();
  }
}

---

## 4. DTO Pattern

### Request DTOs
// Application/DTOs/Requests/Create{Entity}Request.cs
public record Create{Entity}Request(
  string Name,
  string Description,
  decimal Price
);

// Application/DTOs/Requests/Update{Entity}Request.cs
public record Update{Entity}Request(
  string? Name,
  string? Description,
  decimal? Price
);

### Response DTOs
// Application/DTOs/Responses/{Entity}Response.cs
public record {Entity}Response(
  int Id,
  string Name,
  string Description,
  decimal Price,
  DateTime CreatedAt
);

### Paginated Response
// Application/DTOs/Responses/PagedResponse.cs
public record PagedResponse<T>(
  IEnumerable<T> Data,
  int Page,
  int PageSize,
  int TotalCount,
  int TotalPages,
  bool HasNext,
  bool HasPrev
);

### API Response wrapper
// Application/DTOs/Responses/ApiResponse.cs
public record ApiResponse<T>(
  bool Success, T? Data,
  string? Message, IEnumerable<string>? Errors,
  DateTime Timestamp) {
  public static ApiResponse<T> Ok(T data)
    => new(true, data, null, null, DateTime.UtcNow);
  public static ApiResponse<T> Fail(string message)
    => new(false, default, message, null, DateTime.UtcNow);
  public static ApiResponse<T> ValidationFail(IEnumerable<string> errors)
    => new(false, default, "Validation failed", errors, DateTime.UtcNow);
}

---

## 5. Strategy Pattern

### Interface
// Application/Interfaces/Strategies/I{Action}Strategy.cs
public interface IDiscountStrategy {
  decimal Apply(decimal price, int quantity);
  string Name { get; }
}

### Implementations
// Application/Services/Strategies/
public class VipDiscountStrategy : IDiscountStrategy {
  public string Name => "VIP";
  public decimal Apply(decimal price, int quantity) => price * 0.8m;
}

public class BulkDiscountStrategy : IDiscountStrategy {
  public string Name => "Bulk";
  public decimal Apply(decimal price, int quantity)
    => quantity >= 10 ? price * 0.9m : price;
}

public class NoDiscountStrategy : IDiscountStrategy {
  public string Name => "None";
  public decimal Apply(decimal price, int quantity) => price;
}

### Context
// Application/Services/PricingService.cs
public class PricingService {
  private readonly IEnumerable<IDiscountStrategy> _strategies;

  public PricingService(IEnumerable<IDiscountStrategy> strategies)
    => _strategies = strategies;

  public decimal Calculate(decimal price, int quantity, string strategyName) {
    var strategy = _strategies.FirstOrDefault(s => s.Name == strategyName)
      ?? new NoDiscountStrategy();
    return strategy.Apply(price, quantity);
  }
}

### DI Registration
builder.Services.AddScoped<IDiscountStrategy, VipDiscountStrategy>();
builder.Services.AddScoped<IDiscountStrategy, BulkDiscountStrategy>();
builder.Services.AddScoped<PricingService>();

---

## 6. Factory Pattern

### Interface
// Application/Interfaces/Factories/I{Object}Factory.cs
public interface ITokenFactory {
  string CreateAccessToken(User user, IList<string> roles);
  string CreateRefreshToken();
}

public interface IEmailFactory {
  EmailMessage CreateWelcomeEmail(User user);
  EmailMessage CreatePasswordResetEmail(User user, string token);
}

### Implementation
// Application/Services/Factories/TokenFactory.cs
public class TokenFactory : ITokenFactory {
  private readonly IConfiguration _config;
  public TokenFactory(IConfiguration config) => _config = config;

  public string CreateAccessToken(User user, IList<string> roles) {
    var claims = new List<Claim> {
      new(ClaimTypes.NameIdentifier, user.Id),
      new(ClaimTypes.Email, user.Email!)
    };
    claims.AddRange(roles.Select(r => new Claim(ClaimTypes.Role, r)));
    var key = new SymmetricSecurityKey(
      Encoding.UTF8.GetBytes(_config["JWT:Secret"]!));
    var token = new JwtSecurityToken(
      issuer: _config["JWT:Issuer"],
      audience: _config["JWT:Audience"],
      claims: claims,
      expires: DateTime.UtcNow.AddMinutes(60),
      signingCredentials: new SigningCredentials(key, SecurityAlgorithms.HmacSha256)
    );
    return new JwtSecurityTokenHandler().WriteToken(token);
  }

  public string CreateRefreshToken() {
    var bytes = new byte[64];
    RandomNumberGenerator.Fill(bytes);
    return Convert.ToBase64String(bytes);
  }
}

---

## 7. Mediator Pattern (CQRS)

### Installer MediatR
dotnet add {Projet}.Application package MediatR
dotnet add {Projet}.API package MediatR.Extensions.Microsoft.DependencyInjection

### Command (ecriture)
// Application/Commands/Create{Entity}Command.cs
public record Create{Entity}Command(
  string Name, string Description
) : IRequest<{Entity}Response>;

// Application/Commands/Handlers/Create{Entity}CommandHandler.cs
public class Create{Entity}CommandHandler
  : IRequestHandler<Create{Entity}Command, {Entity}Response> {
  private readonly IUnitOfWork _uow;
  private readonly IMapper _mapper;

  public Create{Entity}CommandHandler(IUnitOfWork uow, IMapper mapper) {
    _uow = uow; _mapper = mapper;
  }

  public async Task<{Entity}Response> Handle(
    Create{Entity}Command request, CancellationToken ct) {
    var entity = _mapper.Map<{Entity}>(request);
    await _uow.{Entities}.AddAsync(entity);
    await _uow.SaveChangesAsync();
    return _mapper.Map<{Entity}Response>(entity);
  }
}

### Query (lecture)
// Application/Queries/Get{Entity}ByIdQuery.cs
public record Get{Entity}ByIdQuery(int Id) : IRequest<{Entity}Response>;

// Application/Queries/Handlers/Get{Entity}ByIdQueryHandler.cs
public class Get{Entity}ByIdQueryHandler
  : IRequestHandler<Get{Entity}ByIdQuery, {Entity}Response> {
  private readonly IUnitOfWork _uow;
  private readonly IMapper _mapper;

  public async Task<{Entity}Response> Handle(
    Get{Entity}ByIdQuery request, CancellationToken ct) {
    var entity = await _uow.{Entities}.GetByIdAsync(request.Id)
      ?? throw new NotFoundException(nameof({Entity}), request.Id);
    return _mapper.Map<{Entity}Response>(entity);
  }
}

### Controller avec MediatR
[HttpPost]
public async Task<ActionResult<ApiResponse<{Entity}Response>>> Create(
  Create{Entity}Command command) {
  var result = await _mediator.Send(command);
  return CreatedAtAction(nameof(GetById), new { id = result.Id },
    ApiResponse<{Entity}Response>.Ok(result));
}

### DI Registration
builder.Services.AddMediatR(cfg =>
  cfg.RegisterServicesFromAssembly(typeof(Create{Entity}CommandHandler).Assembly));

---

## 8. Observer Pattern (Domain Events)

### Event de base
// Domain/Events/IDomainEvent.cs
public interface IDomainEvent {
  DateTime OccurredAt { get; }
}

// Domain/Events/{Entity}CreatedEvent.cs
public record {Entity}CreatedEvent(
  int EntityId, string Name, DateTime OccurredAt
) : IDomainEvent;

### Entite avec events
// Domain/Entities/BaseEntity.cs
public abstract class BaseEntity {
  private readonly List<IDomainEvent> _domainEvents = new();
  public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents;
  protected void AddDomainEvent(IDomainEvent @event) => _domainEvents.Add(@event);
  public void ClearDomainEvents() => _domainEvents.Clear();
}

// Domain/Entities/{Entity}.cs
public class {Entity} : BaseEntity {
  public static {Entity} Create(string name) {
    var entity = new {Entity} { Name = name };
    entity.AddDomainEvent(new {Entity}CreatedEvent(entity.Id, name, DateTime.UtcNow));
    return entity;
  }
}

### Handler
// Application/EventHandlers/{Entity}CreatedEventHandler.cs
public class {Entity}CreatedEventHandler
  : INotificationHandler<{Entity}CreatedEvent> {
  private readonly IEmailService _emailService;

  public async Task Handle(
    {Entity}CreatedEvent notification, CancellationToken ct) {
    await _emailService.SendWelcomeEmailAsync(notification.EntityId);
  }
}

---

## 9. Decorator Pattern

### Decorator de cache
// Application/Services/Decorators/Cached{Entity}Service.cs
public class Cached{Entity}Service : I{Entity}Service {
  private readonly I{Entity}Service _inner;
  private readonly IMemoryCache _cache;
  private readonly TimeSpan _expiry = TimeSpan.FromMinutes(5);

  public Cached{Entity}Service(I{Entity}Service inner, IMemoryCache cache) {
    _inner = inner; _cache = cache;
  }

  public async Task<{Entity}Response> GetByIdAsync(int id) {
    var key = $"{entity}_{id}";
    if (_cache.TryGetValue(key, out {Entity}Response? cached))
      return cached!;
    var result = await _inner.GetByIdAsync(id);
    _cache.Set(key, result, _expiry);
    return result;
  }

  // Autres methodes deleguent a _inner
  public Task<PagedResponse<{Entity}Response>> GetAllAsync(
    int page, int size, string? search)
    => _inner.GetAllAsync(page, size, search);

  public Task<{Entity}Response> CreateAsync(Create{Entity}Request request)
    => _inner.CreateAsync(request);

  public Task<{Entity}Response> UpdateAsync(int id, Update{Entity}Request request)
    => _inner.UpdateAsync(id, request);

  public Task DeleteAsync(int id)
    => _inner.DeleteAsync(id);
}

### DI Registration avec Decorator
builder.Services.AddScoped<{Entity}Service>();
builder.Services.AddScoped<I{Entity}Service>(sp => {
  var inner = sp.GetRequiredService<{Entity}Service>();
  var cache = sp.GetRequiredService<IMemoryCache>();
  return new Cached{Entity}Service(inner, cache);
});
builder.Services.AddMemoryCache();

---

## Quand utiliser quel pattern

| Situation | Pattern recommande |
|---|---|
| Acces base de donnees | Repository + Unit of Work |
| Logique metier | Service Pattern |
| Transfert de donnees | DTO Pattern |
| Plusieurs algorithmes interchangeables | Strategy |
| Creation d objets complexes | Factory |
| Projet complexe avec CQRS | Mediator |
| Reactions aux changements | Observer + Domain Events |
| Ajouter cache ou logging sans modifier | Decorator |
