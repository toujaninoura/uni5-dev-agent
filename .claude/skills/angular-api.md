---
name: angular-api
description: Connecter Angular a l API .NET - CORS, proxy, JWT interceptor, services
---

# Skill - Connexion Angular API .NET

## CORS cote .NET (Program.cs)
builder.Services.AddCors(options => {
  options.AddPolicy("AngularApp", policy => {
    policy.WithOrigins("http://localhost:4200")
          .AllowAnyHeader()
          .AllowAnyMethod()
          .AllowCredentials();
  });
});
app.UseCors("AngularApp");

## Service API generique Angular
@Injectable({ providedIn: "root" })
export class ApiService {
  private http = inject(HttpClient);

  get<T>(endpoint: string, params?: any): Observable<ApiResponse<T>> {
    return this.http.get<ApiResponse<T>>(`${environment.apiUrl}${endpoint}`, { params });
  }
  post<T>(endpoint: string, body: any): Observable<ApiResponse<T>> {
    return this.http.post<ApiResponse<T>>(`${environment.apiUrl}${endpoint}`, body);
  }
  put<T>(endpoint: string, body: any): Observable<ApiResponse<T>> {
    return this.http.put<ApiResponse<T>>(`${environment.apiUrl}${endpoint}`, body);
  }
  delete<T>(endpoint: string): Observable<ApiResponse<T>> {
    return this.http.delete<ApiResponse<T>>(`${environment.apiUrl}${endpoint}`);
  }
}

## JWT Interceptor
export const jwtInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (token) {
    req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
  }
  return next(req);
};

## Error Interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) inject(Router).navigate(["/auth/login"]);
      if (error.status === 403) inject(Router).navigate(["/forbidden"]);
      return throwError(() => error);
    })
  );
};

## AuthService
getToken(): string | null { return localStorage.getItem("access_token"); }
isAuthenticated(): boolean {
  const token = this.getToken();
  if (!token) return false;
  const payload = JSON.parse(atob(token.split(".")[1]));
  return payload.exp > Date.now() / 1000;
}

## Ordre de demarrage
Terminal 1 : cd {ProjectName}.API && dotnet run
Terminal 2 : cd {ProjectName}-frontend && ng serve --proxy-config proxy.conf.json
API  : https://localhost:5001/swagger
App  : http://localhost:4200
