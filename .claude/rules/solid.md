# Rules - SOLID

## S - Single Responsibility
Chaque classe a UNE seule responsabilite.
Si tu ne peux pas resumer la classe en une phrase sans "et" -> viole SRP.

INTERDIT :
public class UserService {
  CreateUser()    // logique metier
  GenerateToken() // JWT
  SendEmail()     // email
}

OBLIGATOIRE : un service par responsabilite.

## O - Open/Closed
Ouvert a l extension, ferme a la modification.
Utiliser des interfaces et strategies plutot que des if/switch.

## L - Liskov Substitution
Les sous-classes remplacent leurs parents sans changer le comportement.
Preferer la composition a l heritage.

## I - Interface Segregation
Plusieurs interfaces specifiques > une interface generale.

INTERDIT : IRepository avec 10 methodes dont la moitie inutilisees.
OBLIGATOIRE : IReadRepository + IWriteRepository separes.

## D - Dependency Inversion
Dependre des abstractions, pas des implementations.

INTERDIT :
public class ProductService {
  private readonly ProductRepository _repo = new ProductRepository();
}

OBLIGATOIRE :
public class ProductService {
  private readonly IProductRepository _repo;
  public ProductService(IProductRepository repo) => _repo = repo;
}

Jamais de "new" pour les services et repositories.
Toujours injecter via le constructeur.
