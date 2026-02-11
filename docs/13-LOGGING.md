# 13 - Logging y Monitoreo

> Si no puedes ver que falla, no puedes arreglarlo.

---

## Niveles de Logging

```
NIVEL   | USO                                | PRODUCCION
--------|------------------------------------|-----------
ERROR   | Algo fallo y requiere atencion     | Si
WARN    | Algo inesperado pero no critico    | Si
INFO    | Eventos relevantes del negocio     | Si (moderado)
DEBUG   | Detalle para desarrollo            | No
```

---

## Logging Estructurado

```
MAL (texto plano):
  console.log("Usuario 123 compro producto 456")

BIEN (JSON estructurado):
  logger.info({
    event: "compra_realizada",
    userId: 123,
    productId: 456,
    monto: 29990,
    timestamp: "2026-01-15T10:30:00Z"
  })
```

### Librerias

| Lenguaje | Libreria | Nota |
|----------|----------|------|
| Node.js | Pino | Rapido, JSON por defecto |
| Node.js | Winston | Flexible, multiples transports |
| NestJS | Logger integrado + Pino | Usar PinoLogger para produccion |
| Java | SLF4J + Logback | Estandar en Spring Boot |
| PHP | Monolog | Incluido en Laravel |

### Que Loggear

```
SI:
  - Errores con stack trace
  - Intentos de login fallidos
  - Acciones criticas (pagos, eliminaciones, cambios de permisos)
  - Inicio y detencion de la aplicacion
  - Requests lentos (> 2 segundos)

NO:
  - Passwords o tokens
  - Datos personales sensibles (RUT, tarjetas)
  - Cada request HTTP (usar access log de Nginx para eso)
  - Queries SQL completas con datos de usuario
```

```

---

## Estrategia de Manejo de Errores

### 1. Global Exception Filter (Backend)
Nunca dejes que el usuario vea un "Internal Server Error" crudo de Nginx/Apache.
Captura TODAS las excepciones en un punto central.

**Flujo:**
1.  **Error ocurre:** `DB connection failed`.
2.  **Filter captura:** Transforma a JSON amigable.
3.  **Log:** Registra el stack trace completo con nivel ERROR.
4.  **Respuesta al Usuario:**
    ```json
    {
      "statusCode": 500,
      "message": "Ha ocurrido un error interno. ID de traza: abc-123",
      "timestamp": "..."
    }
    ```

### 2. Redis para Confiabilidad (Colas)
Si una tarea falla (ej: enviar email, procesar video), NO la pierdas.
Usa **Redis** con librerías de colas (BullMQ en Node, Celery en Python, Sidekiq en Ruby).

**Patrón Dead Letter Queue (DLQ):**
*   **Intento 1:** Falla.
*   **Intento 2 (Retry Backoff):** Falla.
*   **Intento 3:** Falla.
*   **Acción:** Mover a cola "Dead" (Muerta) para inspección manual humana. ¡No borres el dato!

---

## Monitoreo de Errores

### Sentry (Recomendado)

```
Gratis: 5K eventos/mes (suficiente para proyectos freelance).

Integracion:
  npm install @sentry/node

  // main.ts (NestJS)
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1,  // 10% de requests para performance
  });

Beneficios:
  - Stack traces completos
  - Agrupacion automatica de errores
  - Alertas por email/Slack
  - Context del usuario y request
  - Source maps para frontend
```

### Alternativas

| Herramienta | Gratis | Pagado |
|-------------|--------|--------|
| Sentry | 5K eventos/mes | $26/mes |
| Highlight.io | 500 sesiones/mes | $50/mes |
| Betterstack (Logtail) | 1GB/mes | $25/mes |
| Self-hosted (Grafana+Loki) | Gratis (infra propia) | - |

---

## Analítica de Producto (Comportamiento de Usuario)

No confundir con Logs de Sistema. Aquí medimos "¿Qué hace el usuario?".

### PostHog (Recomendado)
El "Google Analytics" para desarrolladores modernos. Open Source y auto-hospedable.

*   **Session Replay:** Mira exactamente qué hizo el usuario (como un video). Ideal para reproducir bugs visuales.
*   **Feature Flags:** Activa/Desactiva funciones ("beta") para ciertos usuarios sin re-desplegar.
*   **Heatmaps:** Dónde hacen click.

### Google Analytics 4 (GA4)
*   **Uso:** Estándar de marketing. Necesario si tienes equipo de marketing que vive en el ecosistema Google Ads.
*   **Contra:** Complejo para devs, bloqueado por AdBlockers.

**Estrategia Dual:**
*   **GA4:** Para equipo de Marketing (Fuentes de tráfico).
*   **PostHog:** Para equipo de Producto/Dev (Uso de features).

---

## Monitoreo de Uptime

```
Verificar que el sitio/API esta disponible.

Herramientas:
  - Uptime Robot (gratis, 50 monitores, checks cada 5 min)
  - Better Stack (gratis, paginas de estado bonitas)
  - Cronitor (monitoreo de cron jobs)

Configurar alertas por:
  - Email
  - SMS (si es critico)
  - Webhook a Slack/Discord
```

---

## Metricas de Aplicacion

### Que Medir

```
METRICA                  | POR QUE
-------------------------|------------------------------------------
Tiempo de respuesta API  | Detectar endpoints lentos
Tasa de errores (5xx)    | Detectar problemas en produccion
Uso de memoria/CPU       | Prevenir caidas por recursos
Usuarios activos         | Entender uso real del sistema
Queries lentas           | Optimizar BD
Tasa de conversion       | Medir efectividad (ecommerce)
```

### Stack de Monitoreo (Self-hosted)

```
Para proyectos que lo justifiquen:

Grafana   -> Dashboards y visualizacion
Prometheus -> Recoleccion de metricas
Loki      -> Agregacion de logs

Todo se puede levantar con Docker Compose.
Solo usar si el proyecto es grande o el cliente lo pide.
```

---

## Health Check Endpoint

```typescript
// Implementar en toda API
@Get('health')
healthCheck() {
  return {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV,
  };
}

// Health check con verificacion de BD
@Get('health/ready')
async readinessCheck() {
  try {
    await db.execute(sql`SELECT 1`);
    return { status: 'ok', database: 'connected' };
  } catch {
    throw new ServiceUnavailableException('Database not available');
  }
}
```

Uptime Robot apunta a `/health` para verificar disponibilidad.
