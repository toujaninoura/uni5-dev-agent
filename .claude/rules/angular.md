# Règles Angular 17+ obligatoires

## Version et configuration
- Angular 17+ uniquement (standalone components)
- TypeScript strict mode activé
- ESLint configuré
- Jasmine + Karma pour les tests

## Architecture obligatoire
## Composants
- Standalone components obligatoires (pas de NgModule sauf AppModule)
- Selector en kebab-case : app-product-list
- Template max 50 lignes → extraire en sous-composants
- CSS isolé dans chaque composant
- Jamais de logique métier → déléguer aux services
- Toujours utiliser async pipe dans les templates

INTERDIT :
```typescript
// ❌ Subscribe dans le composant
ngOnInit() {
  this.productService.getProducts().subscribe(p => this.products = p);
}
```

OBLIGATOIRE :
```typescript
// ✅ Observable + async pipe
products$ = this.productService.getProducts();
// Dans le template : *ngFor="let p of products$ | async"
```

## Services
- Un service par ressource API (ProductService, AuthService...)
- Injectés via inject() (Angular 17+)
- HttpClient via HttpClientModule
- Gestion d'erreurs dans le service, pas dans le composant

```typescript
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private apiUrl = `${environment.apiUrl}/api/v1/products`;

  getProducts(params?: PagedRequest): Observable<ApiResponse<PagedResponse<Product>>> {
    return this.http.get<ApiResponse<PagedResponse<Product>>>(this.apiUrl, { params: { ...params } })
      .pipe(catchError(this.handleError));
  }

  private handleError(error: HttpErrorResponse): Observable<never> {
    console.error('API Error:', error);
    return throwError(() => error.error?.message || 'Une erreur est survenue');
  }
}
```

## Intercepteurs obligatoires

### JWT Interceptor
```typescript
export const jwtInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }
  return next(req);
};
```

### Error Interceptor
```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) inject(Router).navigate(['/auth/login']);
      if (error.status === 403) inject(Router).navigate(['/forbidden']);
      return throwError(() => error);
    })
  );
};
```

## Guards
```typescript
export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);
  if (authService.isAuthenticated()) return true;
  return router.createUrlTree(['/auth/login']);
};
```

## Models (interfaces TypeScript)
```typescript
// Toujours typer les réponses API
export interface ApiResponse<T> {
  success: boolean;
  data: T;
  message: string | null;
  errors: string[] | null;
}

export interface PagedResponse<T> {
  data: T[];
  page: number;
  pageSize: number;
  totalCount: number;
  totalPages: number;
  hasNext: boolean;
  hasPrev: boolean;
}

export interface Product {
  id: number;
  name: string;
  price: number;
  stock: number;
  categoryId: number;
  createdAt: string;
}
```

## Routing
```typescript
// Lazy loading obligatoire
export const routes: Routes = [
  { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
  {
    path: 'auth',
    loadChildren: () => import('./features/auth/auth.routes').then(m => m.AUTH_ROUTES)
  },
  {
    path: 'dashboard',
    canActivate: [authGuard],
    loadChildren: () => import('./features/dashboard/dashboard.routes').then(m => m.DASHBOARD_ROUTES)
  }
];
```

## Environnements
```typescript
// environment.ts
export const environment = {
  production: false,
  apiUrl: 'https://localhost:5001'
};

// environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://ton-api.com'
};
```

## CSS custom — conventions
- Variables CSS dans :root
- BEM pour le nommage des classes
- Mobile-first avec breakpoints : 576px, 768px, 992px, 1200px
- Pas de style inline dans les templates

```css
:root {
  --primary: #6c63ff;
  --secondary: #534ab7;
  --success: #00d68f;
  --error: #ff5c5c;
  --text: #333;
  --bg: #f5f5f5;
  --radius: 8px;
  --shadow: 0 2px 8px rgba(0,0,0,0.1);
}
```

## Tests Jasmine obligatoires
- Chaque service : tester tous les appels HTTP
- Chaque composant : tester le rendu et les interactions
- Chaque guard : tester les cas authentifié et non authentifié
- Couverture minimale : 80%

## Interdit
- NgModules sauf AppModule
- Subscribe sans unsubscribe (utiliser takeUntilDestroyed)
- any comme type TypeScript
- Logique métier dans les composants
- Style inline dans les templates
- console.log en production
