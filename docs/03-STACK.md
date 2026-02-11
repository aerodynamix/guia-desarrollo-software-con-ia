# 03 - Stack Tecnologico

> Elige la herramienta correcta para el problema correcto. No uses un canon para matar una mosca.

---

## Stack por Defecto

Si no se especifica lo contrario, usar:

| Capa | Tecnologia | Version |
|------|------------|---------|
| **Frontend** | **Angular** | **20+ (Zoneless)** |
| **Backend** | **NestJS** | **11+** |
| **BD principal** | PostgreSQL | 17+ |
| **Cache** | Redis | 7+ |
| **ORM** | TypeORM o Prisma | Ultima estable |
| **Runtime** | Node.js | 20+ LTS |
| **Deploy** | Docker + Dokploy | VPS |

---

## Matriz de Decision por Tipo de Proyecto

```
PROYECTO             | FRONTEND              | BACKEND           | BD           | RUNTIME
---------------------|-----------------------|-------------------|--------------|--------
Landing Page         | Astro + Tailwind      | No necesario      | No necesario | Node
E-commerce           | Angular / Nuxt 3      | NestJS            | PostgreSQL   | Node
Dashboard            | Angular               | NestJS            | PostgreSQL   | Node
CRM                  | Angular               | NestJS / Spring   | PostgreSQL   | Node/JVM
SaaS                 | Angular / Nuxt 3      | NestJS / Spring   | PostgreSQL   | Node/JVM
POS                  | Electron/Tauri+Angular | NestJS            | SQLite/PgSQL | Node
App Movil            | Flutter / Ionic        | NestJS            | PostgreSQL   | Node
API REST             | -                     | NestJS            | PostgreSQL   | Node
Desktop              | Tauri + Angular        | -                 | SQLite/PgSQL | Nativo
```

---

## Frameworks Frontend

| Framework | Version | Caso de Uso | Prioridad |
|-----------|---------|-------------|-----------|
| **Angular** | **20+** | **Dashboards, SaaS, CRM, E-commerce, Sistemas.** Default (Zoneless). | **1ra opcion** |
| Nuxt.js | 3.x | SSR/SSG con SEO critico (e-commerce publico) | Si SSR es critico |
| Astro | 4.x+ | Landing pages, blogs, sitios estaticos | Solo landing |
| React / Next.js | 19+ / 15+ | Si el equipo ya domina React | Bajo solicitud |
| Tauri | 2.x | Apps de escritorio livianas | Solo desktop |
| Flutter | 3.x | Apps moviles iOS/Android | Solo mobile |

---

## Frameworks Backend

| Framework | Lenguaje | Caso de Uso | Prioridad |
|-----------|----------|-------------|-----------|
| **NestJS** | **TypeScript** | **APIs REST/GraphQL, microservicios, SaaS.** Default. | **1ra opcion** |
| Spring Boot 3+ | Java/Kotlin | Sistemas enterprise, alta concurrencia | Si JVM requerido |
| FastAPI | Python | IA/Data, servicios ligeros | Si Python requerido |
| Express | TypeScript | APIs simples, prototipos rapidos | Solo prototipos |

---

## Bases de Datos

| BD | Tipo | Usar Para | Prioridad |
|----|------|-----------|-----------|
| **PostgreSQL 17+** | Relacional | Produccion, SaaS, sistemas complejos. **Default.** | **1ra opcion** |
| SQLite | Relacional embebida | Apps Electron/Tauri, POS offline | Solo desktop/offline |
| MongoDB | Documental | Logs, datos no estructurados | Bajo solicitud |
| Redis | Cache/Colas | Cache, sesiones, BullMQ | Siempre con PostgreSQL |

### Servicios Managed (Alternativas)
| Servicio | Cuando usar |
|----------|-------------|
| **Neon** | Si solo necesitas BD PostgreSQL robusta sin self-hosting |
| **Supabase** | MVPs rapidos (Auth + BD + Realtime + Storage integrado) |

---

## ORMs

| ORM | Lenguaje | Mejor Para | Prioridad |
|-----|----------|------------|-----------|
| **TypeORM** | TypeScript | Proyectos NestJS, Angular full-stack | **1ra opcion** |
| **Prisma** | TypeScript | Proyectos con esquema declarativo | Alternativa valida |
| DrizzleORM | TypeScript | Proyectos minimalistas, type-safe | Bajo solicitud |
| JPA/Hibernate | Java | Proyectos Spring Boot | Solo JVM |

---

## Lenguajes Principales

| Lenguaje | Usar Para | No Usar Para |
|----------|-----------|--------------|
| **TypeScript** | **Todo proyecto web (front + back).** Default. | Scripts rapidos |
| Java/Kotlin | Sistemas enterprise, Spring Boot | Landing pages |
| Dart | Apps moviles con Flutter | Backend, web |
| Python | Scripts, IA, Backend (FastAPI) | Frontend |
| Rust | CLI, Desktop (Tauri), sistemas criticos | CRUDs simples |

---

## Estilos de API

| Estilo | Cuando Usarlo | Cuando NO Usarlo |
|--------|---------------|------------------|
| **REST** | Default: Simple, cacheable, facil de debuggear | Grafos de datos complejos |
| **GraphQL** | Dashboards complejos con datos anidados | APIs simples, CRUDs basicos |

**Default: REST.** Usar GraphQL solo si el frontend lo demanda explicitamente.

---

## Cliente HTTP (Frontend Angular)

| Herramienta | Recomendacion |
|-------------|---------------|
| **HttpClient (@angular/common/http)** | **Default para Angular. Interceptors, tipado, integrado.** |
| Fetch nativo | Scripts simples sin Angular |
| Axios | Solo proyectos legacy |

---

## Background Jobs y Colas

| Tipo | Herramienta | Caso de Uso |
|------|-------------|-------------|
| Cron Simple | `@nestjs/schedule` | Tareas ligeras: limpiar tmp, resumen diario |
| **Colas (Redis)** | **BullMQ** | Tareas pesadas: emails masivos, procesamiento, webhooks |
| Colas (SQL) | pg-boss | Si no quieres mantener Redis |

**Regla:** Si la tarea tarda > 10 segundos o es critica, usa **Colas** (BullMQ).

---

## Automatizacion y Low-Code

| Herramienta | Tipo | Caso de Uso |
|-------------|------|-------------|
| **n8n** | Workflow | Webhooks, CRMs, Google Sheets, Email. Self-hosted. |
| Windmill | Scripts/UI | Scripts Python/TS como UI interna y cron jobs |

**Regla:** Solo mover datos de A a B? Usa **n8n**. Logica de negocio compleja? Usa **Codigo**.

---

## Deploy (Prioridad)

| Opcion | Costo | Cuando usar |
|--------|-------|-------------|
| **Dokploy en VPS** (1ra) | $5-10/mes | Proyectos medianos/grandes. **Default.** |
| Coolify en VPS (2da) | $5-10/mes | Alternativa a Dokploy |
| Vercel | Gratis - $20/mes | Solo frontend (SSR/SSG) |
| Railway | $5 - $50/mes | Prototipos rapidos |

**VPS recomendados:** Hetzner ($5/mes, mejor precio/rendimiento), DigitalOcean ($24/mes, mejor docs).

---

## UI/UX Frameworks

| Framework CSS | Mejor Para | Prioridad |
|---------------|------------|-----------|
| **Angular Material** | **Proyectos Angular. Default.** | **1ra opcion** |
| TailwindCSS 3+ | Diseno personalizado, utilidades | Alternativa valida |
| PrimeNG | Componentes enterprise Angular | Si set completo necesario |
| CDK (Angular) | Componentes custom accesibles | Base para design systems |

---

## Prueba de APIs

| Herramienta | Tipo | Ventaja |
|-------------|------|---------|
| **Bruno** | Local, Git-friendly | Sin cuenta, archivos en repo. **Default.** |
| Postman | Cloud | Colaboracion en equipo |

---

## Calidad y Formato

- **Formateo:** Respetar formateador existente del proyecto. No cambiar tabulaciones a espacios sin autorizacion.
- **Linter:** ESLint obligatorio. No ignorar "Unused variables". Imports ordenados.
- **Ortografia:** Tildes correctas en documentacion.
- **Prohibido:** Emojis en cualquier contexto tecnico.
