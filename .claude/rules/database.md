# Règles Base de données

## Connection String (appsettings.json)
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database={ProjectName}Db;Trusted_Connection=True;TrustServerCertificate=True;"
}

## AppDbContext
- Hérite de IdentityDbContext<User> si auth JWT activée
- Hérite de DbContext sinon
- Configurations via IEntityTypeConfiguration<T>
- Seed data dans OnModelCreating

## Conventions EF Core obligatoires
- AsNoTracking() sur toutes les requêtes en lecture
- Include() explicite (pas de lazy loading)
- Index sur FK et colonnes filtrées fréquemment
- Precision(18,2) sur tous les décimaux
- HasMaxLength() sur tous les strings
- IsRequired() explicite sur les champs obligatoires
- DeleteBehavior.Restrict par défaut (pas de cascade)

## Migrations
- Une migration = un changement logique
- Nommage : AddProductTable, AddUserIndexes, UpdatePriceColumn
- Toujours vérifier le SQL généré avant d'appliquer
- Jamais modifier une migration déjà appliquée en production

## Seed Data obligatoire
Toujours créer des données de test réalistes :
- 3-5 catégories
- 10-15 produits
- 2 utilisateurs (admin + user standard)
- Rôles : Admin, User

## Commandes
dotnet ef migrations add {Name} --project {Projet}.Infrastructure --startup-project {Projet}.API
dotnet ef database update --project {Projet}.Infrastructure --startup-project {Projet}.API
dotnet ef migrations remove --project {Projet}.Infrastructure --startup-project {Projet}.API
