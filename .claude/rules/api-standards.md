# Rules - API Standards REST

## Routes
- Toujours en kebab-case : /api/v1/product-categories
- Toujours versionnee : /api/v1/
- Ressource au pluriel : /products (pas /product)
- Hierarchie logique : /categories/{id}/products

## Methodes HTTP
- GET    -> lecture (jamais de side effects)
- POST   -> creation -> retourne 201 + ressource creee
- PUT    -> remplacement complet -> retourne 200
- PATCH  -> modification partielle -> retourne 200
- DELETE -> suppression -> retourne 204

## Codes HTTP corrects
200 OK           -> GET reussi, PUT reussi
201 Created      -> POST reussi
204 No Content   -> DELETE reussi
400 Bad Request  -> validation echouee
401 Unauthorized -> token manquant ou invalide
403 Forbidden    -> droits insuffisants
404 Not Found    -> ressource inexistante
409 Conflict     -> doublon
422 Unprocessable-> logiquement invalide
500 Server Error -> erreur interne

## Reponse obligatoire ApiResponse<T>
{
  "success": true,
  "data": {},
  "message": null,
  "errors": null,
  "timestamp": "2024-01-01T00:00:00Z"
}

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

## Interdit
- Verbes dans les routes : /getProducts /createUser
- Routes non versionnees : /api/products
- Retourner une entite directement sans DTO
- GET avec body
- Exposer les IDs sequentiels sans protection
- Lancer des exceptions non gerees
