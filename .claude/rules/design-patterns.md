# Rules - Design Patterns

## Patterns obligatoires dans tout projet

### Repository Pattern
- Un repository par entite
- Interface dans Application, implementation dans Infrastructure
- Jamais appeler DbContext directement depuis un service
- Toujours passer par IRepository

### Unit of Work Pattern
- Un seul SaveChangesAsync() par operation metier
- Jamais SaveChanges() dans les repositories
- Toujours injecter IUnitOfWork dans les services

### Service Pattern
- Un service par domaine metier
- Interface dans Application/Interfaces/
- Implementation dans Application/Services/
- Jamais de logique metier dans les controllers

### DTO Pattern
- Jamais retourner une entite directement
- Request DTO pour les entrees (Create, Update)
- Response DTO pour les sorties
- Utiliser record pour les DTOs (immutabilite)

### Strategy Pattern
- Utiliser quand plusieurs algorithmes font la meme chose
- Exemple : differentes strategies de calcul, de notification
- Interface IStrategy + implementations concretes
- Injecter la strategie via DI

### Factory Pattern
- Utiliser pour creer des objets complexes
- Centraliser la logique de creation
- Exemple : TokenFactory, EmailFactory
- Ne pas instancier directement dans les services

### Mediator Pattern (CQRS)
- Separer les commandes (ecriture) des queries (lecture)
- Commands -> modifier l etat
- Queries -> lire l etat sans modifier
- Utiliser MediatR si projet complexe

### Observer Pattern
- Utiliser pour les evenements domaine
- Exemple : envoyer email apres creation utilisateur
- Domain Events dans les entites
- Handlers dans Application

### Decorator Pattern
- Ajouter des comportements sans modifier la classe
- Exemple : cache, logging, validation sur un service
- Utiliser avec DI pour etre transparent

## Regles generales
- Choisir le pattern le plus simple qui resout le probleme
- Ne pas sur-ingenier -> YAGNI (You Ain t Gonna Need It)
- Un pattern par responsabilite
- Toujours tester les patterns via interfaces mockables
