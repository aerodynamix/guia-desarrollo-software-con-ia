# 00 - Glosario Técnico

> Diccionario de términos y conceptos técnicos usados en Guía de Desarrollo de Software con IA.

---

## Arquitectura y Patrones

### DTO (Data Transfer Object)
Objeto simple usado para transferir datos entre capas o sistemas. No contiene lógica de negocio, solo getters/setters.
```typescript
// DTO - Solo datos
export class CrearUsuarioDto {
  nombre: string;
  email: string;
  password: string;
}
```

### Entity (Entidad)
Representa una tabla de la base de datos. Incluye relaciones, validaciones y puede tener lógica de dominio.
```typescript
// Entity - Mapea a BD
@Entity('usuarios')
export class Usuario {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column()
  nombre: string;
  
  @OneToMany(() => Pedido, pedido => pedido.usuario)
  pedidos: Pedido[];
}
```

### Model (Modelo)
Término genérico. Puede ser Entity, DTO o Value Object según el contexto. En MVC, es la capa de datos.

### Repository (Repositorio)
Patrón que abstrae el acceso a datos. Permite cambiar la BD sin afectar la lógica de negocio.
```typescript
interface UsuarioRepository {
  findById(id: string): Promise<Usuario | null>;
  save(usuario: Usuario): Promise<void>;
}
```

### Service (Servicio)
Contiene la lógica de negocio. Orquesta repositorios y aplica reglas del dominio.

### Controller (Controlador)
Maneja las peticiones HTTP. Valida input, llama servicios y devuelve respuestas.

---

## Autenticación y Seguridad

### JWT (JSON Web Token)
Token firmado que contiene información del usuario. Stateless (no requiere almacenar en servidor).
```
Header.Payload.Signature
eyJ0eXAi...eyJzdWIi...SflKxwRJ
```

### Session (Sesión)
Datos del usuario almacenados en el servidor. Stateful (requiere Redis o BD).

### OAuth 2.0
Protocolo de autorización. Permite login con Google, Facebook, etc.

### CORS (Cross-Origin Resource Sharing)
Mecanismo que permite que un origen (dominio) acceda a recursos de otro origen.

### CSRF (Cross-Site Request Forgery)
Ataque donde un sitio malicioso hace peticiones en nombre del usuario autenticado.

### XSS (Cross-Site Scripting)
Inyección de código JavaScript malicioso en un sitio web.

### SQL Injection
Ataque donde se inyecta código SQL en un input para manipular la base de datos.

---

## Base de Datos

### ORM (Object-Relational Mapping)
Herramienta que mapea objetos de código a tablas de BD (TypeORM, Sequelize, Prisma, Eloquent).

### Migration (Migración)
Script versionado que modifica el schema de la BD (crear tabla, agregar columna, etc.).

### Seed (Seeding)
Poblar la BD con datos de prueba o datos iniciales necesarios.

### Index (Índice)
Estructura que acelera las búsquedas en una columna. Como el índice de un libro.

### Foreign Key (Clave Foránea)
Columna que referencia la clave primaria de otra tabla. Mantiene integridad referencial.

### Soft Delete (Borrado Suave)
Marcar un registro como eliminado sin borrarlo físicamente. Se usa columna `deleted_at`.

### Transaction (Transacción)
Conjunto de operaciones que se ejecutan todas o ninguna (ACID: Atomicidad, Consistencia, Aislamiento, Durabilidad).

---

## Frontend

### SSR (Server-Side Rendering)
El HTML se genera en el servidor por cada petición. Bueno para SEO.

### SSG (Static Site Generation)
El HTML se genera en build time. Ultra rápido, ideal para blogs.

### CSR (Client-Side Rendering)
El HTML se genera en el navegador con JavaScript. Típico de SPAs.

### SPA (Single Page Application)
App de una sola página que se recarga dinámicamente sin recargar la página completa.

### Hydration (Hidratación)
Proceso donde el JavaScript "activa" el HTML estático generado por SSR/SSG.

### Tree Shaking
Eliminación de código no usado durante el bundle. Reduce tamaño del archivo final.

### Code Splitting
Dividir el código en múltiples archivos que se cargan bajo demanda.

### Lazy Loading
Cargar recursos (imágenes, componentes) solo cuando se necesitan.

---

## Estado y Reactividad

### State Management (Gestión de Estado)
Forma de manejar datos compartidos (Angular Signals, NgRx SignalStore, Pinia, Zustand).

### Signals (Señales)
Primitiva reactiva que notifica a los consumidores cuando cambia su valor. Base de Angular 20+.
```typescript
const count = signal(0);
const double = computed(() => count() * 2);
```

### Zoneless (Sin Zonas)
Modo de Angular donde la detección de cambios se dispara explícitamente por Signals, sin usar `zone.js`. Mejora rendimiento y DX.

### Resource API (`rxResource`)
API moderna de Angular para manejar carga de datos asíncronos (http) integrándolos con Signals.

### Local State (Estado Local)
Estado que vive dentro de un componente. Se pierde al desmontar el componente.

### Global State (Estado Global)
Estado compartido entre múltiples componentes. Persiste mientras la app esté abierta.

### Server State (Estado del Servidor)
Datos que vienen del backend. Se manejan con librerías como TanStack Query, SWR.

### Reactive (Reactivo)
Sistema donde los cambios en los datos actualizan automáticamente la UI.

---

## DevOps y Despliegue

### CI/CD (Continuous Integration / Continuous Deployment)
Pipeline automático que prueba y despliega código cada vez que se hace push.

### Docker
Plataforma para empaquetar aplicaciones en contenedores portables.

### Container (Contenedor)
Paquete aislado que incluye la app y todas sus dependencias.

### Image (Imagen)
Plantilla de solo lectura usada para crear contenedores.

### Volume (Volumen)
Almacenamiento persistente para contenedores Docker.

### Reverse Proxy
Servidor que recibe peticiones y las reenvía a otros servidores (Nginx, Traefik).

### Load Balancer
Distribuye tráfico entre múltiples servidores para evitar sobrecarga.

### Blue-Green Deployment
Técnica donde se mantienen dos ambientes (azul y verde). Se cambia el tráfico instantáneamente.

### Canary Deployment
Despliegue gradual. Se expone la nueva versión solo a un % de usuarios.

---

## API y Comunicación

### REST (Representational State Transfer)
Arquitectura de API basada en HTTP. Usa verbos (GET, POST, PUT, DELETE) y recursos (/usuarios, /productos).

### GraphQL
Lenguaje de consulta para APIs. El cliente pide exactamente los datos que necesita.

### Endpoint
URL específica de una API que realiza una acción (ej: POST /api/usuarios).

### Query Parameters
Parámetros en la URL (ej: /productos?page=1&limit=10).

### Path Parameters
Parámetros en el path (ej: /usuarios/:id).

### Body
Datos enviados en el cuerpo de una petición POST/PUT/PATCH.

### Status Code
Código HTTP que indica el resultado (200 OK, 404 Not Found, 500 Internal Server Error).

### Rate Limiting
Limitar el número de peticiones que un cliente puede hacer en un periodo de tiempo.

---

## Testing

### Unit Test (Test Unitario)
Prueba de una función o clase aislada. Rápido y barato.

### Integration Test (Test de Integración)
Prueba de múltiples componentes trabajando juntos (ej: endpoint + BD).

### E2E Test (End-to-End)
Prueba del flujo completo desde la UI hasta la BD. Lento y costoso.

### Mock (Simulacro)
Objeto falso que simula el comportamiento de uno real. Usado en tests.

### Stub
Objeto con respuestas predefinidas. Más simple que un mock.

### Coverage (Cobertura)
Porcentaje de código ejecutado por los tests.

### TDD (Test-Driven Development)
Metodología donde escribes el test antes del código.

---

## Performance

### Lazy Loading
Cargar recursos solo cuando se necesitan.

### Caching (Cacheo)
Almacenar datos en memoria para evitar recalcularlos o pedirlos al servidor.

### CDN (Content Delivery Network)
Red de servidores distribuidos geográficamente que sirven contenido estático rápidamente.

### Throttling
Limitar la frecuencia de ejecución de una función.

### Debouncing
Esperar a que el usuario deje de hacer una acción antes de ejecutar una función.

### Bundle Size
Tamaño total del JavaScript que se envía al navegador.

---

## Otros

### Monorepo
Repositorio único que contiene múltiples proyectos relacionados.

### Polyrepo
Múltiples repositorios, uno por proyecto.

### Technical Debt (Deuda Técnica)
Código de baja calidad que se acumula y dificulta el mantenimiento futuro.

### Refactoring (Refactorización)
Mejorar la estructura del código sin cambiar su funcionalidad.

### Boilerplate
Código repetitivo necesario para configurar un proyecto.

### Linter
Herramienta que analiza código en busca de errores y mal estilo (ESLint, Prettier).

### Transpiler
Herramienta que convierte código de un lenguaje a otro (TypeScript → JavaScript).

### Bundler
Herramienta que agrupa múltiples archivos en uno solo (Webpack, Vite, Rollup).

---

## Siglas Comunes

```
API:    Application Programming Interface
CLI:    Command Line Interface
CRUD:   Create, Read, Update, Delete
DRY:    Don't Repeat Yourself
KISS:   Keep It Simple, Stupid
SOLID:  Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
YAGNI:  You Aren't Gonna Need It
ACID:   Atomicity, Consistency, Isolation, Durability
WCAG:   Web Content Accessibility Guidelines
SEO:    Search Engine Optimization
UI:     User Interface
UX:     User Experience
MVC:    Model-View-Controller
MVP:    Minimum Viable Product
SaaS:   Software as a Service
PaaS:   Platform as a Service
IaaS:   Infrastructure as a Service
GDPR:   General Data Protection Regulation
CCPA:   California Consumer Privacy Act
OWASP:  Open Web Application Security Project
TTL:    Time To Live
JWT:    JSON Web Token
UUID:   Universally Unique Identifier
RBAC:   Role-Based Access Control
2FA:    Two-Factor Authentication
MFA:    Multi-Factor Authentication
VPS:    Virtual Private Server
SSH:    Secure Shell
SSL:    Secure Sockets Layer
TLS:    Transport Layer Security
DNS:    Domain Name System
IP:     Internet Protocol
TCP:    Transmission Control Protocol
HTTP:   Hypertext Transfer Protocol
HTTPS:  HTTP Secure
FTP:    File Transfer Protocol
SMTP:   Simple Mail Transfer Protocol
```

---

> **Nota:** Este glosario se actualizará conforme aparezcan nuevos conceptos en el ecosistema.
