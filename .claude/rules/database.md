# Rules - Database

## Connection String
"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database={ProjectName}Db;Trusted_Connection=True;TrustServerCertificate=True;"
}

## EF Core obligatoire
- AsNoTracking() sur toutes les requetes lecture
- Include() explicite (pas de lazy loading)
- Index sur FK et colonnes filtrees frequemment
- Precision(18,2) sur tous les decimaux
- HasMaxLength() sur tous les strings
- IsRequired() explicite sur les champs obligatoires
- DeleteBehavior.Restrict par defaut
- HasQueryFilter pour le soft delete

## Entites
- Id en int ou Guid
- CreatedAt et UpdatedAt sur toutes les entites
- Soft delete : IsDeleted + DeletedAt
- Jamais de logique dans les entites

## Migrations
- Une migration = un changement logique
- Nommage : Add{Entity}Table, Update{Field}, Add{Entity}Index
- Jamais modifier une migration deja appliquee en production

## Commandes migrations
dotnet ef migrations add {Name} --project {Projet}.Infrastructure --startup-project {Projet}.API
dotnet ef database update --project {Projet}.Infrastructure --startup-project {Projet}.API

## Seed data obligatoire
Toujours creer des donnees de test realistes :
- 3-5 categories
- 10-15 entites principales
- 2 utilisateurs (admin + user standard)
- Roles : Admin, User
