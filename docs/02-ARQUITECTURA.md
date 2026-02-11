# 02 - Arquitectura de Software

> Alta cohesion, bajo acoplamiento. Cada modulo hace lo suyo y se comunica limpiamente.

---

## Principios Arquitectonicos

### Alta Cohesion
Todo lo relacionado a una funcionalidad vive junto.
```
BIEN: /features/productos/ -> components, services, models, routes de productos
MAL:  /controllers/        -> todos los controladores mezclados
```

### Bajo Acoplamiento
Los modulos se comunican por interfaces, no por dependencias directas.
```
BIEN: ProductService recibe un RepositoryInterface (inyeccion de dependencias)
MAL:  ProductService instancia directamente una clase de base de datos
```

---

## Escalabilidad del Software

### 1. Escalabilidad Horizontal (Stateless)
*   **Regla:** El backend NO debe guardar estado en memoria (sesiones, variables globales).
*   **Solución:** Usa Redis para sesiones/cache y JWT para auth.
*   **Beneficio:** Puedes tener 10 instancias de tu API detrás de un Load Balancer.

### 2. Asincronía y Resiliencia (Redis)
*   **Colas de Trabajo:** Operaciones lentas (emails, reportes, IA) van a una cola (BullMQ/Celery).
*   **Retries Automáticos:** Si un servicio externo cae, la cola reintenta automáticamente (Exponential Backoff).
*   **Dead Letter Queues:** Si falla N veces, se guarda para revisión humana. Nada se pierde.

### 3. Base de Datos
*   **Indices:** Todo campo usado en `WHERE`, `ORDER BY` o `JOIN` debe tener índice.
*   **Replicación:** 1 nodo Escritura (Master) + N nodos Lectura (Replicas).
*   **Connection Pooling:** Usa PgBouncer o pool interno del ORM.

### 4. Caching (Niveles)
*   **Browser:** Cache-Control headers para assets estáticos.
*   **CDN (Cloudflare):** Cachea HTML/JSON público en el borde.
*   **App (Redis):** Cachea resultados de queries pesadas o respuestas de API externas.

---

## Arquitectura Limpia (Clean Architecture)

```
+--------------------------------------------------+
|                  PRESENTACION                     |
|        (Componentes, Controladores, DTOs)         |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|               APLICACION / CASOS DE USO           |
|        (Servicios, Logica de Aplicacion)          |
+--------------------------------------------------+
                       |
                       v
+--------------------------------------------------+
|           DOMINIO (Nucleo del Negocio)            |
|      (Entidades, Reglas, Interfaces Puertos)      |
+--------------------------------------------------+
                       ^ (Implementa interfaz)
                       |
+--------------------------------------------------+
|               INFRAESTRUCTURA                     |
|     (BD, APIs externas, Email, Storage, Cache)    |
+--------------------------------------------------+
```
**Nota:** En Hexagonal, Infraestructura *depende* de Dominio (implementando sus interfaces).
El Dominio es puro y no conoce nada del exterior.

---

## Patrones de Diseño Comunes

> **Advertencia:** No uses patrones por usar. Úsalos para resolver problemas específicos.

### 1. Strategy (Estrategia)
*   **Problema:** Tienes múltiples formas de hacer algo (ej: pagar con Paypal, Webpay, Stripe).
*   **Solución:** Interfaz `PaymentProcessor` y clases `PaypalStrategy`, `StripeStrategy`.
*   **Uso:** Cambiar comportamiento en tiempo de ejecución sin llenar de `if/else`.

### 2. Factory (Fabrica)
*   **Problema:** La creación de un objeto es compleja o depende de configuración.
*   **Solución:** Clase `NotificationFactory` que devuelve `EmailNotifier` o `SMSNotifier`.

### 3. Observer (Observador)
*   **Problema:** Algo sucede y varias partes del sistema deben enterarse.
*   **Solución:** Eventos. `UserRegisteredEvent` -> dispara `SendEmailListener` y `CreateWalletListener`.

### 4. Singleton (Inyeccion de Dependencias)
*   **Uso:** Servicios, Conexiones a BD, Logger. Manejado nativamente por los contenedores DI de Angular y NestJS.

---

## Estrategia de Base de Datos

### 1. Identificadores (IDs)
*   **UUID v7 (Recomendado):** Para todas las entidades públicas (Usuarios, Productos, Pedidos).
    *   *Ventaja:* Es ordenable por tiempo (como Serial) pero único globalmente (como UUID v4). Evita enumeración por atacantes.
*   **Serial / BigInt:** Úsalo SOLO para tablas pivote internas o logs masivos donde el espacio es crítico y el ID no se expone.

### 2. Auditoría Básica (Campos Estándar)
Todas las tablas deben tener:
*   `created_at`: Timestamp (UTC).
*   `updated_at`: Timestamp (UTC).
*   `deleted_at`: Timestamp (Nullable) -> Para **Soft Delete**. NUNCA borres datos físicos a menos que sea por GDPR/Ley.

### 3. Auditoría Avanzada (Audit Logs)
Para sistemas financieros, médicos o críticos, implementa una tabla `audit_logs`:

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | UUID | ID del evento |
| `table_name` | VARCHAR | Tabla afectada (ej: `users`) |
| `record_id` | UUID | ID del registro afectado |
| `action` | VARCHAR | CREATE, UPDATE, DELETE |
| `old_value` | JSONB | Valor anterior (snapshot) |
| `new_value` | JSONB | Valor nuevo (snapshot) |
| `changed_by` | UUID | ID del usuario que hizo el cambio |
| `ip_address` | VARCHAR | IP del cliente |
| `created_at` | TIMESTAMP | Fecha del evento |

---

## Estructuras de Carpetas

### Frontend (Angular 19+)

```
src/
  app/
    core/             # Singleton services, guards, interceptors, models globales
      auth/           # Logica especifica de auth
      interceptors/
      services/
      models/
    features/         # Modulos funcionales (lazy loaded)
      dashboard/
        components/
        pages/
        dashboard.routes.ts
      products/
        components/
        pages/
        services/
        models/
        products.routes.ts
    shared/           # Componentes/Pipes/Directivas reutilizables
      components/     # UI Kit (Botones, Inputs, Tablas)
      pipes/
      directives/
      utils/
    app.routes.ts
    app.config.ts
    app.component.ts
  assets/
  styles/
```

### Backend (NestJS)

```
src/
  modules/            # Modulos por dominio
    auth/
      dto/
      guards/
      strategies/
      auth.controller.ts
      auth.service.ts
      auth.module.ts
    products/
      dto/
      entities/
      products.controller.ts
      products.service.ts
      products.module.ts
  common/             # Codigo compartido
    decorators/
    filters/
    interceptors/
    pipes/
    utils/
  config/             # Configuracion (TypeORM, Swagger, Env)
  database/           # Migraciones y Seeds
  main.ts
```

---

## Convenciones de Nombrado

```
Archivos componente:   kebab-case.component.ts (product-list.component.ts)
Archivos servicio:     kebab-case.service.ts (auth.service.ts)
Archivos controlador:  kebab-case.controller.ts
Archivos modelo:       kebab-case.model.ts / kebab-case.entity.ts
Archivos modulo:       kebab-case.module.ts
Archivos ruta:         kebab-case.routes.ts
Carpetas:              kebab-case
Variables/funciones:   camelCase
Clases/Interfaces:     PascalCase
Constantes:            UPPER_SNAKE_CASE
Tablas BD:             snake_case (plural)
Columnas BD:           snake_case
Rutas API:             kebab-case (/api/v1/my-products)
```
