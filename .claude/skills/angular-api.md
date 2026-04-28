# Skill — Connexion Angular ↔ API .NET

## Configuration CORS côté .NET (Program.cs)
```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularApp", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

// Après app.UseRouting()
app.UseCors("AngularApp");
```

## Service Angular générique
```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);

  get<T>(endpoint: string, params?: any): Observable<ApiResponse<T>> {
    return this.http.get<ApiResponse<T>>(
      `${environment.apiUrl}${endpoint}`,
      { params }
    );
  }

  post<T>(endpoint: string, body: any): Observable<ApiResponse<T>> {
    return this.http.post<ApiResponse<T>>(
      `${environment.apiUrl}${endpoint}`,
      body
    );
  }

  put<T>(endpoint: string, body: any): Observable<ApiResponse<T>> {
    return this.http.put<ApiResponse<T>>(
      `${environment.apiUrl}${endpoint}`,
      body
    );
  }

  delete<T>(endpoint: string): Observable<ApiResponse<T>> {
    return this.http.delete<ApiResponse<T>>(
      `${environment.apiUrl}${endpoint}`
    );
  }
}
```

## AuthService complet
```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private api = inject(ApiService);
  private router = inject(Router);
  private readonly TOKEN_KEY = 'access_token';
  private readonly REFRESH_KEY = 'refresh_token';

  login(request: LoginRequest): Observable<AuthResponse> {
    return this.api.post<AuthResponse>('/api/v1/auth/login', request)
      .pipe(
        map(res => res.data),
        tap(auth => this.saveTokens(auth))
      );
  }

  register(request: RegisterRequest): Observable<AuthResponse> {
    return this.api.post<AuthResponse>('/api/v1/auth/register', request)
      .pipe(map(res => res.data), tap(auth => this.saveTokens(auth)));
  }

  logout(): void {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.REFRESH_KEY);
    this.router.navigate(['/auth/login']);
  }

  getToken(): string | null {
    return localStorage.getItem(this.TOKEN_KEY);
  }

  isAuthenticated(): boolean {
    const token = this.getToken();
    if (!token) return false;
    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      return payload.exp > Date.now() / 1000;
    } catch { return false; }
  }

  private saveTokens(auth: AuthResponse): void {
    localStorage.setItem(this.TOKEN_KEY, auth.accessToken);
    localStorage.setItem(this.REFRESH_KEY, auth.refreshToken);
  }
}
```

## Structure d'un projet Fullstack
## Ordre de démarrage en développement
```bash
# Terminal 1 — API .NET
cd ~/projects/{ProjectName}/{ProjectName}.API
dotnet run

# Terminal 2 — Angular
cd ~/projects/{ProjectName}/{ProjectName}-frontend
ng serve --proxy-config proxy.conf.json
```

API  → https://localhost:5001/swagger
App  → http://localhost:4200
