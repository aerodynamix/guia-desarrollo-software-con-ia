# GUÍA DE DESARROLLO DE SOFTWARE CON IA - Sistema Operativo para Agentes

> **Sistema completo de instrucciones para construcción autónoma de software profesional**

---

## PARA AGENTES IA

Este directorio contiene un sistema completo de protocolos, mejores prácticas y guías técnicas para que puedas construir proyectos de software de forma autónoma.

**TU PRIMER PASO:**  
Lee `PROMPT-INICIO.md` COMPLETAMENTE antes de hacer cualquier otra cosa.

---

## ESTRUCTURA DEL SISTEMA

```
guia-desarrollo-software-con-ia/
│
├── PROMPT-INICIO.md              ← COMIENZA AQUÍ (OBLIGATORIO)
│                                   Identidad IA, reglas, protocolo de inicio
│
├── docs/                          ← Documentación especializada (38 archivos)
│   ├── 01-PRINCIPIOS.md          ← SOLID, KISS, DRY
│   ├── 02-ARQUITECTURA.md        ← Clean Architecture, patrones
│   ├── 03-STACK.md               ← Angular 20+ / NestJS 11+
│   ├── 07-SEGURIDAD.md           ← OWASP Top 10
│   ├── 11-DESPLIEGUE.md          ← Dokploy, Coolify, Docker
│   ├── 36-MULTIPLATAFORMA.md     ← Flutter, Tauri, Electron
│   └── 40-MIGRACION.md           ← Guía de Refactorización Legacy
│
├── README.md                      ← Este archivo
├── AI-MANIFEST.md                ← Índice OPTIMIZADO para IA
├── CHANGELOG.md                  ← Historial de versiones

---

### Ubicación
Este sistema debe residir en la raíz de tu proyecto o en una carpeta `docs/` dedicada.
El usuario te indicará la ubicación exacta si no es estándar.

---

### NIVEL 1: PROTOCOLO DE INTERROGATORIO

Después de leer `PROMPT-INICIO.md`, ejecuta el protocolo de interrogatorio:

1. Pregunta tipo de proyecto (Landing, E-commerce, SaaS, POS, etc.)
2. Pregunta escala (MVP, Mediano, Enterprise)
3. Sugiere stack tecnológico
4. Pregunta entorno del usuario (Linux/macOS/Windows)

### NIVEL 2: LECTURA SEGÚN TIPO DE PROYECTO

Basándote en las respuestas, lee SOLO los archivos relevantes:

#### Landing Page
```
docs/01-PRINCIPIOS.md
docs/03-STACK.md (Astro, Angular 20+)
docs/05-UI-UX.md
docs/11-DESPLIEGUE.md (Dokploy, Vercel)
docs/24-SEO.md
```

#### E-commerce
```
docs/01-PRINCIPIOS.md
docs/02-ARQUITECTURA.md
docs/03-STACK.md (Angular 20+, NestJS, PostgreSQL)
docs/05-UI-UX.md
docs/07-SEGURIDAD.md (OWASP, PCI DSS)
docs/11-DESPLIEGUE.md (Dokploy)
docs/19-PAGOS.md (Stripe, Flow)
docs/21-FEATURES.md
```

#### SaaS / Sistema Completo
```
docs/01-PRINCIPIOS.md
docs/02-ARQUITECTURA.md
docs/03-STACK.md
docs/04-ESTRUCTURA.md
docs/07-SEGURIDAD.md
docs/11-DESPLIEGUE.md
docs/13-LOGGING.md
docs/14-RENDIMIENTO.md
docs/17-SAAS.md
docs/18-MONETIZACION.md
```

#### Punto de Venta (POS)
```
docs/01-PRINCIPIOS.md
docs/03-STACK.md (Electron, Tauri)
docs/11-DESPLIEGUE.md (Instaladores desktop)
docs/21-FEATURES.md (Hardware)
docs/36-MULTIPLATAFORMA.md
```

#### API Backend
```
docs/01-PRINCIPIOS.md
docs/02-ARQUITECTURA.md
docs/03-STACK.md (NestJS, FastAPI)
docs/07-SEGURIDAD.md
docs/08-TESTING.md
docs/11-DESPLIEGUE.md
docs/13-LOGGING.md
```

#### Aplicación Móvil
```
docs/03-STACK.md (Flutter, React Native)
docs/05-UI-UX.md (Mobile-first)
docs/36-MULTIPLATAFORMA.md
```

---

## TECNOLOGÍAS SOPORTADAS

### Frontend
- **Landing Pages:** Astro
- **E-commerce:** Angular 20+ (SSR)
- **SaaS/Dashboard:** Angular 20+
- **Móvil:** Flutter
- **Desktop:** Tauri (1ra opción), Electron (2da opción)

### Backend
- **API REST:** NestJS (recomendado)
- **Microservicios:** NestJS
- **Serverless:** FastAPI (Python)

### Base de Datos
- **Transaccional:** PostgreSQL 17 (recomendado)
- **NoSQL:** MongoDB
- **Caché:** Redis

### Deployment (PRIORIDAD)
1. **Dokploy** en VPS ($5-10/mes) - RECOMENDADO
2. Coolify en VPS ($5-10/mes) - Alternativa
3. Vercel (solo frontend)
4. Railway ($5-50/mes)

---

## STACK RECOMENDADO POR TIPO DE PROYECTO

### Landing Page
```
- Framework: Astro
- CSS: TailwindCSS
- Deploy: Vercel (gratis) o Dokploy
- Tiempo: 3-5 días
- Costo: $300,000 - $800,000 CLP
```

### E-commerce
```
- Frontend: Angular 20+ (SSR)
- Backend: NestJS
- BD: PostgreSQL 17 + Redis
- Pagos: Stripe + Flow (Chile)
- Deploy: Dokploy en VPS
- Tiempo: 8-10 semanas
- Costo: $3,500,000 - $5,000,000 CLP
```

### SaaS
```
- Frontend: Angular 20+
- Backend: NestJS (microservicios)
- BD: PostgreSQL (multi-tenant)
- Caché: Redis
- Pagos: Stripe Billing
- Deploy: Dokploy en VPS
- Tiempo: 12-20 semanas
- Costo: $10,000,000 - $25,000,000 CLP
```

### POS (Punto de Venta)
```
- Desktop: Tauri + Vue 3
- Backend: NestJS
- BD: SQLite (local) + PostgreSQL (servidor)
- Hardware: USB Serial (pistola), ESCPOS (impresora)
- Deploy: Instalador .exe/.dmg
- Tiempo: 8-12 semanas
- Costo: $4,000,000 - $8,000,000 CLP
```

---

## REGLAS INVIOLABLES

### PROHIBICIONES
1. **CERO EMOJIS** - En código, commits, docs, logs
2. **CERO DATOS REALES** - Solo sintéticos/anonimizados
3. **CERO REGRESIONES** - No toques código funcional
4. **CERO DEPENDENCIAS INNECESARIAS** - Justifica cada paquete

### OBLIGACIONES
1. **SEEDING** - Scripts con `@faker-js/faker`
2. **TESTS** - Colección Bruno/Postman
3. **DIAGRAMAS** - Mermaid para lógica compleja
4. **COMMITS** - `tipo(ámbito): descripción (#issue)`
5. **REPORTES** - Tabla de métricas al finalizar

---

## DOCUMENTOS CLAVE

| Categoría | Archivos | Descripción |
|-----------|----------|-------------|
| **Core** | 01-PRINCIPIOS.md<br>02-ARQUITECTURA.md<br>03-STACK.md | Fundamentos técnicos |
| **Desarrollo** | 04-ESTRUCTURA.md<br>05-UI-UX.md<br>06-GIT.md | Organización y estilo |
| **Seguridad** | 07-SEGURIDAD.md | OWASP Top 10, JWT, CORS |
| **Deploy** | 11-DESPLIEGUE.md<br>12-INFRAESTRUCTURA.md | Dokploy, Docker, VPS |
| **Negocio** | 16-COSTOS.md<br>20-LEGAL.md | Estimaciones, contratos |
| **Debugging** | 25-DEBUGGING.md<br>32-ERROR-HANDLING.md | Resolución de problemas |
| **Multiplataforma** | 36-MULTIPLATAFORMA.md | Flutter, Tauri, Electron |

---

## EJEMPLO DE FLUJO DE TRABAJO

### Usuario dice: "Necesito un e-commerce para vender ropa"

**TU RESPUESTA:**

1. **Ejecutar protocolo de interrogatorio:**
   ```
   "Entiendo que necesitas un e-commerce. Déjame hacerte unas preguntas:
   
   1. ¿Qué escala tiene el proyecto?
      A) MVP (lanzar rápido, 4-6 semanas)
      B) Mediano (escalable, 8-12 semanas)
      C) Enterprise (alta disponibilidad, 12+ semanas)
   
   2. ¿Tienes preferencia de stack o deseas que recomiende?"
   ```

2. **Usuario responde:** "Mediano, recomienda tú"

3. **Lees los archivos relevantes:**
   - `docs/01-PRINCIPIOS.md`
   - `docs/02-ARQUITECTURA.md`
   - `docs/03-STACK.md`
   - `docs/05-UI-UX.md`
   - `docs/07-SEGURIDAD.md`
   - `docs/11-DESPLIEGUE.md`
   - `docs/19-PAGOS.md`

4. **Propones stack:**
   ```
   "Recomiendo:
   
   FRONTEND: Angular 20+ (SSR) + Angular Material + TailwindCSS
   BACKEND: NestJS (TypeScript modular) + PostgreSQL 17 + Redis
   PAGOS: Stripe (internacional) + Flow (Chile)
   DEPLOY: Dokploy en VPS Hetzner ($10/mes)
   
   ESTIMACIÓN:
   - Tiempo: 8-10 semanas
   - Costo: $3,500,000 - $5,000,000 CLP
   
   Features incluidas:
   - Catálogo de productos con búsqueda
   - Carrito de compras
   - Checkout seguro
   - Panel de administración
   - Gestión de inventario
   - Reportes de ventas
   
   ¿Procedes con este stack?"
   ```

5. **Usuario aprueba:** "Sí, adelante"

6. **Generas código** siguiendo principios SOLID, Clean Architecture, OWASP Top 10

7. **Al finalizar sesión, reportas:**
   ```
   | Métrica | Valor |
   |---------|-------|
   | Archivos Creados | 45 |
   | Líneas de Código | +3,200 |
   | Complejidad | 4/5 |
   
   Trabajo realizado:
   - Estructura base del proyecto
   - Módulo de autenticación (JWT)
   - Módulo de productos (CRUD)
   - Configuración de BD (PostgreSQL)
   
   Próximo paso: Implementar carrito de compras
   ```

---

## PRINCIPIOS FUNDAMENTALES

### SOLID
- **S** - Single Responsibility
- **O** - Open/Closed
- **L** - Liskov Substitution
- **I** - Interface Segregation
- **D** - Dependency Inversion

### KISS
- Keep It Simple, Stupid
- La solución más simple es usualmente la mejor

### DRY
- Don't Repeat Yourself
- Abstrae código duplicado

### YAGNI
- You Aren't Gonna Need It
- No construyas funcionalidades especulativas

---

## SEGURIDAD OWASP TOP 10

1. **Inyección SQL** - Usar prepared statements
2. **XSS** - Sanitizar inputs
3. **CSRF** - Tokens CSRF
4. **Autenticación** - JWT con expiración corta
5. **Exposición de datos** - Encriptar datos sensibles
6. **Control de acceso** - RBAC (Role-Based Access Control)
7. **Configuración incorrecta** - Hardening del servidor
8. **Deserialización** - Validar y sanitizar
9. **Componentes vulnerables** - Mantener deps actualizadas
10. **Logging** - Logs sin datos sensibles

**Detalles:** `docs/07-SEGURIDAD.md`

---

## LICENCIA

MIT License - Úsalo libremente.

---

## CONTRIBUCION

¿Quieres mejorar esta guía? ¡Bienvenido!

Consulta [CONTRIBUTING.md](./CONTRIBUTING.md) para conocer las normas y el flujo de trabajo.

---

## CONTACTO Y SOPORTE

**¿Necesitas ayuda?**
- Lee `PROMPT-INICIO.md` primero
- Consulta el archivo relevante en `docs/`
- Ejecuta el protocolo de interrogatorio

**AGENTE IA: LEE `PROMPT-INICIO.md` ANTES DE CUALQUIER TAREA.**

---

**VERSIÓN:** 1.0
**ÚLTIMA ACTUALIZACIÓN:** 10 de Febrero, 2026
