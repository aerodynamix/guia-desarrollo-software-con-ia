# 32 - Manejo Avanzado de Errores

> Los errores van a suceder. La diferencia entre software amateur y profesional es cómo los manejas.

---

## Filosofía de Errores

### Principios Fundamentales
1. **Fail Fast:** Detecta errores lo antes posible
2. **Fail Gracefully:** Cuando falle, no rompas la experiencia completa
3. **Fail Loudly (Dev) / Fail Silently (Prod):** Logs detallados en desarrollo, mensajes amigables en producción

---

## Tipos de Errores

### 1. Errores de Validación (400)
El usuario envió datos inválidos.

**Qué hacer:**
- Mostrar mensaje específico con `MatSnackBar` o `FormControl` errors.
- NO registrar como error crítico (es comportamiento esperado).

```typescript
// Backend (NestJS)
throw new BadRequestException({
  message: 'Datos inválidos',
  errors: { email: 'Ya registrado' }
});

// Frontend (Angular)
this.authService.login(data).subscribe({
  error: (err) => {
    if (err.status === 400) {
      this.snackBar.open('Datos inválidos', 'Cerrar');
    }
  }
});
```

### 2. Errores de Autorización (401/403)
El usuario no tiene permisos.

```typescript
// HttpInterceptor (Angular)
intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
  return next.handle(req).pipe(
    catchError(error => {
      if (error.status === 401) {
        this.auth.logout();
        this.router.navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
}
```

### 3. Errores de Servidor (500)
Algo falló en el backend.

**Qué hacer:**
- Registrar error completo en Sentry.
- Mostrar mensaje genérico al usuario: "Ha ocurrido un error inesperado".

---

## Error Handling Global (Angular)

En Angular, usamos `ErrorHandler` para capturar errores no manejados.

```typescript
// global-error-handler.ts
import { ErrorHandler, Injectable, Injector, NgZone } from '@angular/core';
import { MatSnackBar } from '@angular/material/snack-bar';
import * as Sentry from '@sentry/angular';

@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(private injector: Injector, private zone: NgZone) {}

  handleError(error: any) {
    // 1. Loggear en Sentry (Prod)
    Sentry.captureException(error);
    console.error('Error Global:', error);

    // 2. Notificar al usuario (usar NgZone para asegurar que corra en Angular)
    this.zone.run(() => {
      const snackBar = this.injector.get(MatSnackBar);
      snackBar.open('Ha ocurrido un error inesperado', 'Cerrar', { duration: 5000 });
    });
  }
}

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    { provide: ErrorHandler, useClass: GlobalErrorHandler }
  ]
};
```

---

## Logging de Errores (Sentry)

```typescript
// main.ts
Sentry.init({
  dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
  integrations: [
    Sentry.browserTracingIntegration(),
    Sentry.replayIntegration(),
  ],
  tracesSampleRate: 1.0, 
});
```

---

## Patrones de Backend (NestJS)

### 1. Custom Exceptions
```typescript
export class UserNotFoundException extends NotFoundException {
  constructor(userId: string) {
    super(`Usuario ${userId} no encontrado`);
  }
}
```

### 2. Global Exception Filter
Transforma cualquier error en un formato JSON estándar.

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: ctx.getRequest().url,
    });
  }
}
```

---

## Checklist de Errores

- [ ] Todos los `subscribe` tienen manejo de error (`error: (err) => ...`).
- [ ] Interceptor captura 401/403 globalmente.
- [ ] Sentry activado en producción.
- [ ] Backend nunca devuelve stack traces en producción.
