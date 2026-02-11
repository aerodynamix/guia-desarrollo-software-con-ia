# 23 - Documentación de Proyecto

> Todo proyecto entregado debe incluir documentacion. Minima pero suficiente.
> Dos audiencias: el desarrollador que mantiene y el cliente que usa.

### Herramientas AI para Docs
*   **Mintlify:** Lee tu código y genera la documentación automáticamente.
*   **Cursor:** Puede escribir el README si le das el contexto.

---

## 4. Portal de Documentación (Docs as Code)

Para proyectos Medianos/Grandes, el `README.md` no es suficiente. Se debe crear un sitio estático de documentación.

### Estrategia
Tratar la documentación como código: Markdown en repositorio, versionado, y desplegado automáticamente.

### Herramientas Recomendadas
| Herramienta | Basado en | Mejor Para |
|-------------|-----------|------------|
| **VitePress** | Vue.js | Proyectos Vue/Nuxt. Minimalista, rapidísimo. Estándar de Vue. |
| **Starlight** | Astro | Proyectos agnósticos. Muy flexible, excelente performance. |
| **Docusaurus** | React | Proyectos React. Muy maduro, usado por Meta. |

### Estructura del Sitio
Diferenciar claramente las audiencias:
1.  **Tutoriales:** "Learning by doing" (Paso a paso para nuevos devs).
2.  **Guías:** "How-to" (Cómo implementar Auth, Cómo desplegar).
3.  **Referencia:** API Specs (Scalar/Swagger embed), Diccionario de Datos.
4.  **Explicaciones:** Arquitectura, Decisiones de Diseño (ADRs).

---

## Documentacion Tecnica (Para el Desarrollador)

Esta documentacion vive DENTRO del repositorio. Sirve para que cualquier
desarrollador (humano o IA) pueda entender, instalar y modificar el proyecto.

### README.md (Obligatorio en TODO proyecto)

```markdown
# Nombre del Proyecto

Descripcion de una linea: que hace este sistema.

## Stack

- Frontend: [framework + version]
- Backend: [framework + version]
- BD: [motor + version]
- Runtime: [Node/Bun/JVM version]

## Requisitos Previos

- Node.js 20+ / Java 17+ / PHP 8.2+
- PostgreSQL 17+ / MySQL 8+
- Docker (opcional pero recomendado)

## Instalacion

1. Clonar: `git clone [url]`
2. Copiar variables: `cp .env.example .env`
3. Configurar .env con valores reales
4. Instalar dependencias: `npm install`
5. Ejecutar migraciones: `npm run migrate`
6. Ejecutar seeders: `npm run seed`
7. Iniciar: `npm run dev`

## Scripts

| Comando | Descripcion |
|---------|-------------|
| `npm run dev` | Desarrollo con hot reload |
| `npm run build` | Compilar para produccion |
| `npm run start` | Iniciar en produccion |
| `npm run test` | Ejecutar tests |
| `npm run lint` | Verificar estilo de codigo |
| `npm run migrate` | Ejecutar migraciones BD |
| `npm run seed` | Cargar datos iniciales |

## Variables de Entorno

Ver `.env.example` para la lista completa. Variables criticas:

| Variable | Descripcion | Ejemplo |
|----------|-------------|---------|
| DATABASE_URL | Conexion a BD | postgresql://user:pass@localhost:5432/db |
| JWT_SECRET | Secret para tokens | (generar con openssl rand -hex 32) |
| PORT | Puerto del servidor | 3000 |

## Estructura del Proyecto

(Arbol de carpetas principales con descripcion de 1 linea cada una)

## API

Documentacion interactiva en `/api/docs` (Swagger).
Coleccion Bruno en `/bruno`.

## Despliegue

Ver `docker-compose.prod.yml` o la guia en `docs/deploy.md`.
```

### Documentacion Adicional (proyectos medianos/grandes)

```
docs/
  deploy.md        -> Guia paso a paso de despliegue
  architecture.md  -> Diagrama de arquitectura y decisiones tecnicas
  api.md           -> Notas sobre endpoints no cubiertas por Swagger
  database.md      -> Diagrama ER, decisiones de esquema, indices
```

Solo crear estos archivos si el proyecto lo amerita. No crear por crear.

### Diagramas (cuando sean necesarios)

Usar formato Mermaid (renderiza en GitHub/GitLab):

```markdown
## Diagrama de Arquitectura

    ```mermaid
    graph LR
      A[Cliente/Browser] --> B[Nginx]
      B --> C[Frontend Nuxt]
      B --> D[API NestJS]
      D --> E[PostgreSQL]
      D --> F[Redis Cache]
      D --> G[Storage S3/R2]
    ```

## Diagrama de Base de Datos

    ```mermaid
    erDiagram
      USUARIO ||--o{ PEDIDO : realiza
      PEDIDO ||--|{ DETALLE_PEDIDO : contiene
      PRODUCTO ||--o{ DETALLE_PEDIDO : incluido_en
      USUARIO {
        uuid id PK
        string email
        string password_hash
        string rol
      }
      PEDIDO {
        uuid id PK
        uuid usuario_id FK
        decimal total
        string estado
      }
    ```

## Diagrama de Flujo (Procesos)

    ```mermaid
    flowchart TD
      A[Usuario agrega producto] --> B{Tiene cuenta?}
      B -->|Si| C[Ir a checkout]
      B -->|No| D[Registrarse / Login]
      D --> C
      C --> E[Seleccionar metodo de pago]
      E --> F[Procesar pago]
      F -->|Exitoso| G[Confirmar pedido]
      F -->|Fallido| H[Mostrar error + reintentar]
      F -->|Exitoso| G[Confirmar pedido]
      F -->|Fallido| H[Mostrar error + reintentar]
    ```

## Diagrama de Clases (UML)

    ```mermaid
    classDiagram
      class Usuario {
        +UUID id
        +String email
        +login()
        +register()
      }
      class Pedido {
        +UUID id
        +Date fecha
        +calcularTotal()
      }
      Usuario "1" --> "*" Pedido : realiza
    ```

## Diagrama de Secuencia (Interacción)

    ```mermaid
    sequenceDiagram
      participant U as Usuario
      participant F as Frontend
      participant A as API
      participant D as DB
      U->>F: Clic en "Pagar"
      F->>A: POST /orders
      A->>D: INSERT Order
      D-->>A: Order ID
      A-->>F: 201 Created
      F-->>U: Mostrar "Pago Exitoso"
    ```

## Carta Gantt (Planificación)

    ```mermaid
    gantt
      title Planificación de Proyecto MVP
      dateFormat  YYYY-MM-DD
      section Backend
      Diseño BD       :a1, 2026-03-01, 3d
      API Auth        :a2, after a1, 5d
      API Productos   :after a2, 5d
      section Frontend
      Mockups UI      :2026-03-01, 5d
      Integración     :after a2, 5d
    ```
```

---

## Documentacion Funcional (Para el Cliente)

Esta documentacion se entrega AL CLIENTE. No es tecnica.
Puede ser un PDF, un sitio estatico simple, o un documento compartido.

### Manual de Usuario

```markdown
# Manual de Usuario - [Nombre del Sistema]

## Acceso al Sistema
- URL: https://[dominio]
- Ingresar email y contrasena proporcionados.
- Si olvido su contrasena, usar "Recuperar contrasena".

## Funcionalidades

### [Nombre del Modulo]
Descripcion breve de que hace.

**Como usar:**
1. Ir a [seccion]
2. Hacer clic en [boton]
3. Completar el formulario
4. Guardar

(Incluir capturas de pantalla si es posible)

### Reportes
- Como generar un reporte
- Como exportar a PDF/Excel
- Filtros disponibles

## Preguntas Frecuentes

**P: No puedo iniciar sesion**
R: Verificar que el email es correcto. Usar "Recuperar contrasena" si es necesario.

**P: Como agrego un nuevo [elemento]?**
R: Ir a [seccion] > clic en "Agregar" > completar formulario > Guardar.

## Soporte
- Email: [tu email de soporte]
- Horario: Lunes a Viernes, 9:00 - 18:00
- Tiempo de respuesta: 24-48 horas habiles
```

### Que Incluir Segun Tipo de Proyecto

```
PROYECTO         | README | API Docs | Manual Usuario | Deploy Guide
-----------------|--------|----------|----------------|-------------
Landing Page     | Si     | No       | No             | Basico
Ecommerce        | Si     | Si       | Si             | Si
Dashboard        | Si     | Si       | Si (breve)     | Si
CRM              | Si     | Si       | Si             | Si
POS              | Si     | Si       | Si             | Si
SaaS             | Si     | Si       | Si             | Si
App Movil        | Si     | Si       | Si (en-app)    | Si (stores)
```

---

## Changelog (Registro de Cambios)

Para proyectos con mantenimiento activo, mantener un CHANGELOG.md:

```markdown
# Changelog

## [1.2.0] - 2026-03-15
### Agregado
- Exportacion de reportes a Excel
- Filtro por fecha en listado de ventas

### Corregido
- Error en calculo de IVA con descuento aplicado

## [1.1.0] - 2026-02-20
### Agregado
- Modulo de notificaciones email

### Cambiado
- Rediseno de la pantalla de login
```
