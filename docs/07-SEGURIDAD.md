# 07 - Seguridad Técnica

> La seguridad no es una fase final. Se implementa desde la primera línea de código.

---

## Filosofía: Security by Design (Seguridad por Diseño)

En Guía de Desarrollo de Software con IA, la seguridad no es un parche; es parte de la arquitectura inicial:
1. **Mínimo Privilegio:** Cada servicio, usuario o proceso tiene solo los permisos que necesita para funcionar.
2. **Defensa en Profundidad:** Múltiples capas de seguridad (App, Red, BD, OS).
3. **Superficie de Ataque Mínima:** Desactiva puertos, plugins y dependencias que no uses.
4. **Validación Total:** "Nunca confíes, siempre verifica". Valida todo lo que entre al sistema desde el exterior.

---

## OWASP Top 10 - Mitigación Práctica

### 1. Inyeccion SQL / NoSQL

```
NUNCA concatenar valores directamente en queries.

MAL:
  const query = `SELECT * FROM users WHERE email = '${email}'`;

BIEN:
  const query = 'SELECT * FROM users WHERE email = $1';
  db.query(query, [email]);

MEJOR:
  Usar ORM (TypeORM, Prisma) que parametriza automaticamente.
```

### 2. Cross-Site Scripting (XSS)

```
- Angular escapa el HTML por defecto.
- NUNCA usar `[innerHTML]` sin sanitizar con DomSanitizer.
- Implementar Content-Security-Policy (CSP) en headers.
```

### 3. Cross-Site Request Forgery (CSRF)

```
- Angular HttpClient soporta XSRF token automaticamente si el backend envia la cookie XSRF-TOKEN.
- En SPAs: usar header personalizado + cookie SameSite.
- Configurar cookies con: SameSite=Strict o Lax, HttpOnly, Secure.
```

### 4. Autenticación Rota

```
- Usar bcrypt/argon2 para hashear passwords (NUNCA MD5 o SHA1).
- Implementar rate limiting en login (ej: 5 intentos por minuto).
- JWT: expiracion corta (15min access, 7d refresh).
- Refresh tokens en cookie HttpOnly, NO en localStorage.
```

---

## Autenticacion y Sesiones (Detalle)

### JWT (JSON Web Tokens)

```
FLUJO ESTANDAR:
  1. Usuario envia credenciales (POST /auth/login)
  2. Backend valida, genera access_token (15min) + refresh_token (7-30 dias)
  3. access_token se envia en header: Authorization: Bearer <token>
  4. refresh_token se almacena en cookie HttpOnly Secure SameSite=Strict
  5. Cuando access_token expira, interceptor captura 401 y usa refresh_token para renovar
  6. Si refresh_token expira, usuario debe re-autenticarse
```

### Token Interceptor (Angular)

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getAccessToken();
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }
    return next.handle(req).pipe(
      catchError(error => {
        if (error.status === 401) {
          return this.authService.refreshToken().pipe(
            switchMap(newToken => {
              const newReq = req.clone({
                 setHeaders: { Authorization: `Bearer ${newToken}` }
              });
              return next.handle(newReq);
            })
          );
        }
        return throwError(() => error);
      })
    );
  }
}
```

### Implementacion por Framework

```
NestJS:
  @nestjs/jwt + @nestjs/passport + passport-jwt
  Guards: AuthGuard('jwt'), RolesGuard
  Decoradores: @UseGuards(), @Roles('admin')

Spring Boot:
  Spring Security + jjwt o nimbus-jose-jwt
  SecurityFilterChain + JwtAuthenticationFilter
```

---

## Seguridad en la Interacción con IA (Privacidad)

Para proteger la integridad del desarrollador y del cliente:
1. **Anonimización:** NUNCA pegues datos reales de clientes (nombres, correos, RUT/DNI) en chats con asistentes de IA.
2. **Secretos:** Está prohibido compartir `.env`, API Keys o certificados reales.
3. **Propiedad Intelectual:** La IA es una herramienta de asistencia; el código sensible del core del negocio debe ser manejado con cautela.

---

## Configuracion CORS

```typescript
// NestJS
const corsOptions = {
  origin: ['https://midominio.com'],  // NUNCA usar '*' en produccion
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization'],
};
```

---

## Variables de Entorno

```
REGLAS:
1. NUNCA commitear archivos .env
2. Siempre incluir .env.example con las variables sin valores reales
3. Validar que todas las variables requeridas existen al iniciar la app
4. Usar librerias de validacion: joi, zod (NestJS ConfigModule validation)

Ejemplo .env.example:
  DATABASE_URL=postgresql://user:password@localhost:5432/dbname
  JWT_SECRET=cambiar-este-valor
  PORT=3000
```

---

## Validacion de Datos

```
REGLA: Validar en AMBOS lados (frontend + backend).
El frontend valida para UX. El backend valida para SEGURIDAD.

Frontend:  Angular Reactive Forms Validators
Backend:   DTOs con class-validator (NestJS)

Validar:
  - Tipo de dato
  - Longitud minima/maxima
  - Formato (email, telefono, RUT)
  - Regex para caracteres permitidos
```

### Ejemplo Angular Validator
```typescript
this.loginForm = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [Validators.required, Validators.minLength(8)]]
});
```

---

## Almacenamiento de Tokens

```
BIEN:                              MAL:
Cookie HttpOnly + Secure           localStorage (accesible por XSS)
Cookie SameSite=Strict             sessionStorage (accesible por XSS)
                                   URL query params (visible en logs)
```

---

## Rate Limiting

Implementar en:
- Login: 5 intentos / minuto / IP
- API publica: 100 requests / minuto / IP
- Registro: 3 cuentas / hora / IP

Herramientas: `@nestjs/throttler` (NestJS), `rate-limiter-flexible`.
