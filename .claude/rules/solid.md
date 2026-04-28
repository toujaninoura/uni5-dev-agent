# Principes SOLID obligatoires

## S — Single Responsibility Principle
Chaque classe a UNE seule responsabilité.

INTERDIT :
public class UserService
{
    public async Task<User> CreateAsync(...)  { } // ← logique métier
    public string GenerateToken(User user)    { } // ← génération JWT
    public void SendWelcomeEmail(User user)   { } // ← envoi email
}

OBLIGATOIRE :
- UserService      → logique métier utilisateurs uniquement
- JwtService       → génération et validation tokens uniquement
- EmailService     → envoi emails uniquement

Règle pratique : si tu ne peux pas résumer la classe en une phrase
sans le mot "et", elle viole SRP.

## O — Open/Closed Principle
Ouvert à l'extension, fermé à la modification.

INTERDIT :
public class DiscountService
{
    public decimal Calculate(string type, decimal price)
    {
        if (type == "VIP") return price * 0.8m;
        if (type == "Student") return price * 0.9m;
        // Ajouter un nouveau type = modifier cette classe
    }
}

OBLIGATOIRE :
public interface IDiscountStrategy
{
    decimal Apply(decimal price);
}
public class VipDiscount : IDiscountStrategy
{
    public decimal Apply(decimal price) => price * 0.8m;
}
public class StudentDiscount : IDiscountStrategy
{
    public decimal Apply(decimal price) => price * 0.9m;
}
// Nouveau type = nouvelle classe, pas de modification

## L — Liskov Substitution Principle
Les sous-classes doivent pouvoir remplacer leurs classes parentes.

INTERDIT :
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
}
public class Square : Rectangle
{
    // Casse LSP car Width = Height toujours
    public override int Width { set { base.Width = base.Height = value; } }
}

OBLIGATOIRE :
- Préférer la composition à l'héritage
- Une interface commune plutôt qu'une hiérarchie fragile
- Si override change le comportement attendu → mauvais design

## I — Interface Segregation Principle
Plusieurs interfaces spécifiques valent mieux qu'une interface générale.

INTERDIT :
public interface IRepository<T>
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
    Task<IEnumerable<T>> SearchAsync(string query);   // pas tous en ont besoin
    Task<int> CountAsync();                            // pas tous en ont besoin
    Task BulkInsertAsync(IEnumerable<T> entities);    // pas tous en ont besoin
}

OBLIGATOIRE :
public interface IReadRepository<T>
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
}
public interface IWriteRepository<T>
{
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(int id);
}
public interface ISearchRepository<T>
{
    Task<IEnumerable<T>> SearchAsync(string query);
}
public interface IRepository<T> : IReadRepository<T>, IWriteRepository<T> { }

## D — Dependency Inversion Principle
Dépendre des abstractions, pas des implémentations.

INTERDIT :
public class ProductService
{
    private readonly ProductRepository _repo; // ← classe concrète
    public ProductService()
    {
        _repo = new ProductRepository(); // ← new interdit
    }
}

OBLIGATOIRE :
public class ProductService : IProductService
{
    private readonly IProductRepository _repo; // ← interface
    public ProductService(IProductRepository repo) // ← injection
    {
        _repo = repo;
    }
}

Règle : jamais de "new" pour les services et repositories
Toujours injecter via le constructeur
