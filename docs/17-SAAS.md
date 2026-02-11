# 17 - Arquitectura SaaS

> Guia para desarrollar y vender software como servicio a clientes.

---

## Modelo de Negocio SaaS

### Que es SaaS para un Freelancer

Desarrollas un sistema UNA vez y lo vendes/alquilas a MULTIPLES clientes.
Cada cliente accede a su propia instancia o comparte infraestructura (multitenancy).

### Tipos de Negocio: ¿Venta o Arriendo?

#### 1. Arriendo (SaaS) - Recomendado
*   **Modelo:** "Spotify".
*   **Código:** Es TUYO. Nunca se entrega el fuente.
*   **Cobro:** Mensual/Anual recurrente (MRR).
*   **Ventaja:** Ingreso pasivo acumulable. Valuación de empresa alta (5x-10x ARR).

#### 2. Venta (Licencia One-Time) - "Vender el producto"
*   **Modelo:** "Adobe Photoshop CS6" (Antiguo) o Venta de Script (CodeCanyon).
*   **Código:** Puede ser entregado (Self-hosted por el cliente) o binario ofuscado.
*   **Cobro:** Pago único alto + Soporte anual opcional (15-20% del valor inicial).
*   **Ventaja:** Cash flow inmediato grande.
*   **Desventaja:** Tienes que vender de nuevo cada mes para comer.

### Tipos de Monetizacion

```
MODELO              | DESCRIPCION                         | EJEMPLO
--------------------|-------------------------------------|---------------------------
Suscripcion mensual | Pago recurrente por uso             | $15.000 - $150.000 CLP/mes
Licencia unica      | Pago unico + soporte opcional       | $500.000 - $5.000.000 CLP
Freemium            | Gratis limitado + planes pagados    | Free + Pro + Enterprise
Por uso             | Cobro por transaccion/recurso       | $100 CLP por factura emitida
White label         | Vendes con marca del cliente        | Precio premium
```

### Planes Tipicos

```
PLAN         | CARACTERISTICAS                          | PRECIO SUGERIDO CLP
-------------|------------------------------------------|---------------------
Basico       | 1 usuario, funciones core, soporte email | $15.000 - $30.000/mes
Profesional  | 5 usuarios, reportes, integraciones      | $40.000 - $80.000/mes
Empresarial  | Usuarios ilimitados, soporte prioritario | $100.000 - $250.000/mes
```

---

## Arquitectura Multitenancy

### Estrategias

```
ESTRATEGIA              | AISLAMIENTO | COSTO INFRA | COMPLEJIDAD
------------------------|-------------|-------------|------------
BD separada por tenant  | Alto        | Alto        | Media
Schema separado (PgSQL) | Medio-Alto  | Medio       | Media
Tabla compartida + ID   | Bajo        | Bajo        | Baja
```

**Recomendacion freelance:** Tabla compartida con `tenant_id` para empezar.
Migrar a schema separado si el proyecto crece.

### Implementacion Basica (Tabla Compartida)

```
Cada tabla tiene columna: tenant_id UUID NOT NULL

Middleware/Guard que:
1. Extrae tenant del JWT o subdominio
2. Inyecta tenant_id en TODAS las queries
3. Filtra TODOS los resultados por tenant_id

CRITICO: Nunca permitir que un tenant vea datos de otro.
```

---

## Funcionalidades Core de un SaaS

```
MODULO                 | DESCRIPCION
-----------------------|---------------------------------------------
Auth + Roles           | Registro, login, recuperar password, RBAC
Onboarding             | Wizard de configuracion inicial
Dashboard              | Metricas clave del tenant
Gestion de usuarios    | CRUD usuarios del tenant, roles
Facturacion            | Planes, cobros, historial (Stripe, MercadoPago)
Notificaciones         | Email transaccional, in-app, push
Configuracion          | Branding del tenant, preferencias
Reportes               | Exportar datos (PDF, Excel, CSV)
Audit log              | Registro de acciones del usuario
API publica (opcional) | Para integraciones de terceros
```

---

## Infraestructura Recomendada para SaaS

### Stack Recomendado

```
Frontend:       Nuxt 3 + TailwindCSS (SSR para SEO en landing)
Backend:        NestJS + DrizzleORM
BD:             PostgreSQL (Neon o VPS propio)
Auth:           JWT + Refresh tokens propios, o Auth0/Clerk
Pagos:          Stripe (internacional) o MercadoPago (LATAM)
Email:          Resend, Mailgun o Amazon SES
Storage:        Cloudflare R2, S3, o MinIO (self-hosted)
CDN:            Cloudflare (gratis)
Monitoreo:      Sentry (errores), Uptime Robot (disponibilidad)
```

### Infraestructura por Etapa

```
ETAPA         | INFRA                              | COSTO APROX USD/MES
--------------|------------------------------------|--------------------
MVP (0-50)    | Vercel + Railway + Neon             | $0-25
Crecimiento   | VPS Hetzner 4GB + Docker            | $10-20
Escala        | 2 VPS + Load Balancer + BD managed   | $40-100
Empresa       | Kubernetes o Cloud managed           | $100+
```

---

## Servicios en la Nube por Funcion

### Backend y Hosting

| Servicio | Tipo | Precio Base |
|----------|------|-------------|
| Hetzner Cloud | VPS | $4/mes |
| DigitalOcean | VPS | $6/mes |
| Railway | PaaS | $5/mes |
| Render | PaaS | $7/mes |
| AWS EC2 | IaaS | $8+/mes |

### Base de Datos

| Servicio | Tipo | Precio Base |
|----------|------|-------------|
| Neon | PostgreSQL serverless | Gratis / $19/mes |
| Supabase | PostgreSQL + extras | Gratis / $25/mes |
| PlanetScale | MySQL serverless | Gratis / $29/mes |
| AWS RDS | BD managed | $15+/mes |
| VPS propio | PostgreSQL Docker | Incluido en VPS |

### Email Transaccional

| Servicio | Gratis/mes | Pagado |
|----------|-----------|--------|
| Resend | 3.000 emails | $20/mes |
| Mailgun | 1.000 emails | $15/mes |
| Amazon SES | 62.000 (desde EC2) | $0.10/1000 |
| Brevo (Sendinblue) | 300 emails/dia | $9/mes |

### Storage (Archivos)

| Servicio | Precio |
|----------|--------|
| Cloudflare R2 | 10GB gratis, luego $0.015/GB |
| AWS S3 | $0.023/GB |
| MinIO (self-hosted) | Gratis (open source) |

### Sistemas Operativos para VPS

```
Recomendado:  Ubuntu Server 22.04 LTS o 24.04 LTS
Alternativa:  Debian 12 (mas estable, menos actualizaciones)
Avanzado:     Alpine Linux (ultra liviano para contenedores)
```

---

## Vender SaaS a Clientes

### Modelo de Oferta para Freelancer

```
1. SETUP INICIAL
   - Configuracion del sistema: $200.000 - $500.000 CLP
   - Personalizacion de marca: $100.000 - $300.000 CLP
   - Migracion de datos: $100.000 - $200.000 CLP

2. SUSCRIPCION MENSUAL (al cliente)
   - Incluye: hosting, mantenimiento, soporte, actualizaciones
   - Precio: $30.000 - $150.000 CLP/mes segun funcionalidades

3. DESARROLLO A MEDIDA
   - Funcionalidades adicionales: $30.000 - $60.000 CLP/hora
```

### Que Incluir en Contrato

```
- Alcance exacto de funcionalidades
- SLA (disponibilidad garantizada, ej: 99.5%)
- Soporte: canales (email, WhatsApp), horarios, tiempos de respuesta
- Propiedad del codigo (tuya como SaaS, o del cliente si es a medida)
- Condiciones de cancelacion
- Politica de datos y backups
```
