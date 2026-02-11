# AI Manifest (Ruta de Lectura Optimizada)

> **Instrucción para Agentes de IA:**
> Este archivo define el orden LÓGICO en el que debes consumir la documentación para entender el proyecto "Guía de Desarrollo de Software con IA".
> NO leas los archivos en orden alfabético. Sigue esta ruta.

---

## 1. Contexto y Reglas Base (Lectura Obligatoria)

Estos archivos definen QUÉ estamos construyendo y CÓMO debes comportarte.

1.  **`PROMPT-INICIO.md`** (System Prompt): Tu configuración inicial. Define el stack (Angular 20/NestJS 11) y tu personalidad.
2.  **`03-STACK.md`**: Tecnologías permitidas. Si no está aquí, no lo uses sin preguntar.
3.  **`02-ARQUITECTURA.md`**: Dónde poner los archivos. Estructura de carpetas Monorepo/Polyrepo.
4.  **`00-GLOSARIO.md`**: Definiciones de términos clave (Signals, Zoneless, Resource).

## 2. Desarrollo Activo (Referencia Constante)

Consulta estos archivos según la tarea que estés realizando.

### Frontend (Angular 20+)
*   **Crear Componentes:** Lee `04-ESTRUCTURA.md` (patrón de carpetas) y `05-UI-UX.md` (diseño).
*   **Estado / Datos:** Lee `31-ESTADO.md` (Signals, rxResource, NgRx).
*   **Formularios/Validación:** Lee `07-SEGURIDAD.md` y `32-ERROR-HANDLING.md`.

### Backend (NestJS 11+)
*   **Crear Endpoints:** Lee `02-ARQUITECTURA.md` (Módulos/Controladores).
*   **Base de Datos:** Lee `03-STACK.md` (Prisma/TypeORM) y `14-RENDIMIENTO.md`.
*   **Seguridad:** Lee `07-SEGURIDAD.md` (Guards, JWT).

## 3. Calidad y Operaciones (Antes de cerrar tarea)

*   **Tests:** `08-TESTING.md` (Jest/Vitest/Playwright).
*   **Git:** `06-GIT.md` (Conventional Commits).
*   **Deploy:** `11-DESPLIEGUE.md` (Dokploy/Docker).

## 4. Tareas Especiales

*   **Migrar Proyecto Legacy:** Lee `40-MIGRACION.md`.
*   **Accesibilidad:** Lee `09-ACCESIBILIDAD.md`.
*   **Multi-idioma:** Lee `10-I18N.md`.
*   **Real-time:** Lee `33-REALTIME.md`.

---

## Mapa de Archivos (Índice Completo)

| Archivo | Propósito | Prioridad IA |
|---------|-----------|--------------|
| `PROMPT-INICIO.md` | **System Prompt** | Alta |
| `AI-MANIFEST.md` | Este archivo | Alta |
| `00-GLOSARIO.md` | Diccionario Técnico | Media |
| `01-PRINCIPIOS.md` | Filosofía | Baja |
| `02-ARQUITECTURA.md` | Estructura Proyecto | Alta |
| `03-STACK.md` | Tecnologías | Alta |
| `04-ESTRUCTURA.md` | Árbol de Carpetas | Alta |
| `05-UI-UX.md` | Diseño/Componentes | Media |
| `06-GIT.md` | Versionado | Media |
| `07-SEGURIDAD.md` | Auth/Validación | Alta |
| `08-TESTING.md` | QA/Tests | Media |
| `09-ACCESIBILIDAD` | A11y | Baja (Contextual) |
| `10-I18N.md` | Multi-idioma | Baja (Contextual) |
| `11-DESPLIEGUE.md` | DevOps/Docker | Media |
| `12-INFRA...` | Hardware (Legacy) | Baja |
| `13-LOGGING.md` | Logs/Sentry | Media |
| `14-RENDIMIENTO` | Performance | Media |
| `15-BACKUPS.md` | Respaldos | Baja |
| `21-FEATURES.md` | Utils (PDF/Email) | Baja (Contextual) |
| `31-ESTADO.md` | State Mgmt | Alta |
| `32-ERROR...` | Error Handling | Alta |
| `33-REALTIME.md` | WebSockets/SSE | Baja (Contextual) |
| `40-MIGRACION.md` | Guía Refactor | Alta (Si aplica) |
