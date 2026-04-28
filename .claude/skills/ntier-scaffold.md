---
name: ntier-scaffold
description: Creer la structure complete N-Tier avec tous les patterns - Repository, UoW, Service, DTO
---

# Skill - N-Tier Scaffold Complet

## Patterns a implementer dans l ordre

### 1. Domain - Entite de base
// {Projet}.Domain/Entities/BaseEntity.cs
public abstract class BaseEntity {
  public int Id { get; set; }
  public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
  public DateTime? UpdatedAt { get; set; }
  public bool IsDeleted { get; set; } = false;
  public DateTime? DeletedAt { get; set; }
}

// {Projet}.Domain/Entities/{Entity}.cs
public class {Entity} : BaseEntity {
  public string Name { get; set; } = string.Empty;
  // autres proprietes
}

### 2. Domain - Exceptions
// {Projet}.Domain/Exceptions/AppException.cs
public abstract class AppException : Exception {
  public int StatusCode { get; }
  protected AppException(string message, int statusCode)
    : base(message) => StatusCode = statusCode;
}
public class NotFoundException : AppException {
  public NotFoundException(string name, object key)
    : base($"{name} with key {key} was not found.", 404) {}
}
public class BusinessException : AppException {
  public BusinessException(string message) : base(message, 400) {}
}
public class UnauthorizedException : AppException {
  public UnauthorizedException(string message) : base(message, 401) {}
}
public class ConflictException : AppException {
  public ConflictException(string message) : base(message, 409) {}
}

### 3. Application - Interfaces Repository
// {Projet}.Application/Interfaces/IRepository.cs
public interface IRepository<T> where T : class {
  Task<T?> GetByIdAsync(int id);
  Task<IEnumerable<T>> GetAllAsync();
  Task<T> AddAsync(T entity);
  Task UpdateAsync(T entity);
  Task DeleteAsync(int id);
}

// {Projet}.Application/Interfaces/I{Entity}Repository.cs
public interface I{Entity}Repository : IRepository<{Entity}> {
  Task<IEnumerable<{Entity}>> SearchAsync(string query, int page, int size);
  Task<bool> ExistsAsync(int id);
}

// {Projet}.Application/Interfaces/IUnitOfWork.cs
public interface IUnitOfWork : IDisposable {
  I{Entity}Repository {Entities} { get; }
  // autres repositories
  Task<int> SaveChangesAsync();
}

### 4. Application - DTOs
// {Projet}.Application/DTOs/Requests/Create{Entity}Request.cs
public record Create{Entity}Request(
  string Name,
  // autres champs
);

// {Projet}.Application/DTOs/Responses/{Entity}Response.cs
public record {Entity}Response(
  int Id,
  string Name,
  DateTime CreatedAt
  // autres champs
);

// {Projet}.Application/DTOs/Responses/PagedResponse.cs
public record PagedResponse<T>(
  IEnumerable<T> Data,
  int Page,
  int PageSize,
  int TotalCount,
  int TotalPages,
  bool HasNext,
  bool HasPrev
);

// {Projet}.Application/DTOs/Responses/ApiResponse.cs
public record ApiResponse<T>(
  bool Success,
  T? Data,
  string? Message,
  IEnumerable<string>? Errors,
  DateTime Timestamp
) {
  public static ApiResponse<T> Ok(T data)
    => new(true, data, null, null, DateTime.UtcNow);
  public static ApiResponse<T> Fail(string message)
    => new(false, default, message, null, DateTime.UtcNow);
  public static ApiResponse<T> ValidationFail(IEnumerable<string> errors)
    => new(false, default, "Validation failed", errors, DateTime.UtcNow);
}

### 5. Application - Validators
// {Projet}.Application/Validators/Create{Entity}RequestValidator.cs
public class Create{Entity}RequestValidator : AbstractValidator<Create{Entity}Request> {
  public Create{Entity}RequestValidator() {
    RuleFor(x => x.Name)
      .NotEmpty().WithMessage("Le nom est obligatoire")
      .MaximumLength(100).WithMessage("Le nom ne peut pas depasser 100 caracteres");
  }
}

### 6. Application - Mappings
// {Projet}.Application/Mappings/MappingProfile.cs
public class MappingProfile : Profile {
  public MappingProfile() {
    CreateMap<{Entity}, {Entity}Response>();
    CreateMap<Create{Entity}Request, {Entity}>();
  }
}

### 7. Application - Service
// {Projet}.Application/Interfaces/I{Entity}Service.cs
public interface I{Entity}Service {
  Task<{Entity}Response> GetByIdAsync(int id);
  Task<PagedResponse<{Entity}Response>> GetAllAsync(int page, int size, string? search);
  Task<{Entity}Response> CreateAsync(Create{Entity}Request request);
  Task<{Entity}Response> UpdateAsync(int id, Update{Entity}Request request);
  Task DeleteAsync(int id);
}

// {Projet}.Application/Services/{Entity}Service.cs
public class {Entity}Service : I{Entity}Service {
  private readonly IUnitOfWork _uow;
  private readonly IMapper _mapper;
  private readonly ILogger<{Entity}Service> _logger;

  public {Entity}Service(IUnitOfWork uow, IMapper mapper, ILogger<{Entity}Service> logger) {
    _uow = uow; _mapper = mapper; _logger = logger;
  }

  public async Task<{Entity}Response> GetByIdAsync(int id) {
    _logger.LogInformation("Getting {Entity} {Id}", typeof({Entity}).Name, id);
    var entity = await _uow.{Entities}.GetByIdAsync(id)
      ?? throw new NotFoundException(nameof({Entity}), id);
    return _mapper.Map<{Entity}Response>(entity);
  }

  public async Task<{Entity}Response> CreateAsync(Create{Entity}Request request) {
    _logger.LogInformation("Creating {Entity}", typeof({Entity}).Name);
    var entity = _mapper.Map<{Entity}>(request);
    await _uow.{Entities}.AddAsync(entity);
    await _uow.SaveChangesAsync();
    return _mapper.Map<{Entity}Response>(entity);
  }
}

### 8. Infrastructure - DbContext
// {Projet}.Infrastructure/Data/AppDbContext.cs
public class AppDbContext : IdentityDbContext<User> {
  public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}
  public DbSet<{Entity}> {Entities} => Set<{Entity}>();

  protected override void OnModelCreating(ModelBuilder builder) {
    base.OnModelCreating(builder);
    builder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
  }
}

### 9. Infrastructure - Configuration EF Core
// {Projet}.Infrastructure/Data/Configurations/{Entity}Configuration.cs
public class {Entity}Configuration : IEntityTypeConfiguration<{Entity}> {
  public void Configure(EntityTypeBuilder<{Entity}> builder) {
    builder.HasKey(e => e.Id);
    builder.Property(e => e.Name).IsRequired().HasMaxLength(100);
    builder.HasQueryFilter(e => !e.IsDeleted);
    builder.HasIndex(e => e.Name);
  }
}

### 10. Infrastructure - Repository
// {Projet}.Infrastructure/Repositories/{Entity}Repository.cs
public class {Entity}Repository : I{Entity}Repository {
  private readonly AppDbContext _context;
  public {Entity}Repository(AppDbContext context) => _context = context;

  public async Task<{Entity}?> GetByIdAsync(int id)
    => await _context.{Entities}.AsNoTracking().FirstOrDefaultAsync(e => e.Id == id);

  public async Task<IEnumerable<{Entity}>> GetAllAsync()
    => await _context.{Entities}.AsNoTracking().ToListAsync();

  public async Task<{Entity}> AddAsync({Entity} entity) {
    await _context.{Entities}.AddAsync(entity);
    return entity;
  }

  public async Task UpdateAsync({Entity} entity) {
    _context.{Entities}.Update(entity);
  }

  public async Task DeleteAsync(int id) {
    var entity = await _context.{Entities}.FindAsync(id);
    if (entity != null) {
      entity.IsDeleted = true;
      entity.DeletedAt = DateTime.UtcNow;
    }
  }

  public async Task<IEnumerable<{Entity}>> SearchAsync(string query, int page, int size)
    => await _context.{Entities}.AsNoTracking()
      .Where(e => e.Name.Contains(query))
      .Skip((page - 1) * size).Take(size)
      .ToListAsync();
}

### 11. Infrastructure - UnitOfWork
// {Projet}.Infrastructure/Repositories/UnitOfWork.cs
public class UnitOfWork : IUnitOfWork {
  private readonly AppDbContext _context;
  public I{Entity}Repository {Entities} { get; }

  public UnitOfWork(AppDbContext context, I{Entity}Repository {entities}) {
    _context = context;
    {Entities} = {entities};
  }

  public async Task<int> SaveChangesAsync()
    => await _context.SaveChangesAsync();

  public void Dispose() => _context.Dispose();
}

### 12. API - Controller
// {Projet}.API/Controllers/{Entity}Controller.cs
[ApiController]
[Route("api/v1/[controller]")]
[Authorize]
public class {Entity}Controller : ControllerBase {
  private readonly I{Entity}Service _service;
  public {Entity}Controller(I{Entity}Service service) => _service = service;

  [HttpGet]
  public async Task<ActionResult<ApiResponse<PagedResponse<{Entity}Response>>>> GetAll(
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 10,
    [FromQuery] string? search = null)
  {
    var result = await _service.GetAllAsync(page, pageSize, search);
    return Ok(ApiResponse<PagedResponse<{Entity}Response>>.Ok(result));
  }

  [HttpGet("{id}")]
  public async Task<ActionResult<ApiResponse<{Entity}Response>>> GetById(int id) {
    var result = await _service.GetByIdAsync(id);
    return Ok(ApiResponse<{Entity}Response>.Ok(result));
  }

  [HttpPost]
  public async Task<ActionResult<ApiResponse<{Entity}Response>>> Create(Create{Entity}Request request) {
    var result = await _service.CreateAsync(request);
    return CreatedAtAction(nameof(GetById), new { id = result.Id },
      ApiResponse<{Entity}Response>.Ok(result));
  }

  [HttpPut("{id}")]
  public async Task<ActionResult<ApiResponse<{Entity}Response>>> Update(int id, Update{Entity}Request request) {
    var result = await _service.UpdateAsync(id, request);
    return Ok(ApiResponse<{Entity}Response>.Ok(result));
  }

  [HttpDelete("{id}")]
  public async Task<ActionResult> Delete(int id) {
    await _service.DeleteAsync(id);
    return NoContent();
  }
}

### 13. API - Middleware Global Exception
// {Projet}.API/Middlewares/GlobalExceptionMiddleware.cs
public class GlobalExceptionMiddleware {
  private readonly RequestDelegate _next;
  private readonly ILogger<GlobalExceptionMiddleware> _logger;

  public GlobalExceptionMiddleware(RequestDelegate next, ILogger<GlobalExceptionMiddleware> logger) {
    _next = next; _logger = logger;
  }

  public async Task InvokeAsync(HttpContext context) {
    try { await _next(context); }
    catch (AppException ex) {
      _logger.LogWarning(ex, "Business exception");
      context.Response.StatusCode = ex.StatusCode;
      await context.Response.WriteAsJsonAsync(ApiResponse<object>.Fail(ex.Message));
    }
    catch (Exception ex) {
      _logger.LogError(ex, "Unexpected error");
      context.Response.StatusCode = 500;
      await context.Response.WriteAsJsonAsync(ApiResponse<object>.Fail("Internal server error"));
    }
  }
}

### 14. API - Program.cs
var builder = WebApplication.CreateBuilder(args);

// DbContext
builder.Services.AddDbContext<AppDbContext>(options =>
  options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"),
    b => b.MigrationsAssembly("{Projet}.Infrastructure")));

// Identity + JWT
builder.Services.AddIdentity<User, IdentityRole>(options => {
  options.Password.RequiredLength = 8;
  options.User.RequireUniqueEmail = true;
})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
  .AddJwtBearer(options => {
    options.TokenValidationParameters = new TokenValidationParameters {
      ValidateIssuerSigningKey = true,
      IssuerSigningKey = new SymmetricSecurityKey(
        Encoding.UTF8.GetBytes(builder.Configuration["JWT:Secret"]!)),
      ValidateIssuer = true,
      ValidIssuer = builder.Configuration["JWT:Issuer"],
      ValidateAudience = true,
      ValidAudience = builder.Configuration["JWT:Audience"],
      ValidateLifetime = true,
      ClockSkew = TimeSpan.Zero
    };
  });

// AutoMapper + FluentValidation
builder.Services.AddAutoMapper(typeof(MappingProfile));
builder.Services.AddValidatorsFromAssemblyContaining<Create{Entity}RequestValidator>();
builder.Services.AddFluentValidationAutoValidation();

// Repositories + Services
builder.Services.AddScoped<I{Entity}Repository, {Entity}Repository>();
builder.Services.AddScoped<IUnitOfWork, UnitOfWork>();
builder.Services.AddScoped<I{Entity}Service, {Entity}Service>();

// Swagger avec JWT
builder.Services.AddSwaggerGen(c => {
  c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme {
    Name = "Authorization", Type = SecuritySchemeType.ApiKey,
    Scheme = "Bearer", BearerFormat = "JWT", In = ParameterLocation.Header
  });
  c.AddSecurityRequirement(new OpenApiSecurityRequirement {{
    new OpenApiSecurityScheme {
      Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" }
    }, Array.Empty<string>()
  }});
});

var app = builder.Build();
app.UseMiddleware<GlobalExceptionMiddleware>();
app.UseSwagger(); app.UseSwaggerUI();
app.UseHttpsRedirection();
app.UseCors("AngularApp");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.Run();
