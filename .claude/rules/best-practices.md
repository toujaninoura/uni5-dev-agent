# Rules - Best Practices

## Logging
- ILogger sur tous les services
- Logger les actions importantes : creation, modification, suppression
- Logger les erreurs avec le contexte complet
- Jamais logger de mots de passe ou tokens

## Reponses HTTP standardisees
- 200 OK      -> GET reussi, PUT reussi
- 201 Created -> POST reussi
- 204 No Content -> DELETE reussi
- 400 Bad Request -> validation echouee
- 401 Unauthorized -> token manquant
- 403 Forbidden -> droits insuffisants
- 404 Not Found -> ressource inexistante
- 409 Conflict -> doublon
- 500 Server Error -> erreur interne

## Pagination obligatoire sur GET liste
Request  : ?page=1&pageSize=10&search=keyword&sortBy=name&sortDir=asc
Response :
{
  "data": [],
  "page": 1,
  "pageSize": 10,
  "totalCount": 0,
  "totalPages": 0,
  "hasNext": false,
  "hasPrev": false
}

## Documentation
- Swagger active avec descriptions XML
- Commentaires XML sur tous les endpoints publics
- README.md a la racine avec instructions de demarrage

## Clean Code
- Methodes max 20 lignes
- Pas de magic numbers
- Nommage explicite
- Pas de code commente
- Un niveau d abstraction par methode
