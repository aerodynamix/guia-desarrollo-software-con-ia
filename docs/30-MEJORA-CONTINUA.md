# 30 - Mejora Continua y Retrospectiva

> "El software nunca se termina, solo se abandona."
> Esta guía define cómo evolucionar el sistema y la relación con el cliente después del lanzamiento.

---

## 1. Retrospectiva Técnica (Post-Mortem)

Después de cada hito importante o entrega final, el desarrollador (y la IA) deben analizar:
- **¿Qué salió bien?** (Stack elegido, automatizaciones efectivas).
- **¿Qué salió mal?** (Gaps de seguridad, scope creep no detectado).
- **¿Qué podemos mejorar?** (Nuevas reglas para el Master Prompt, mejores snippets).

---

## 2. Refactorización Proactiva

No esperes a que el código huela mal.
- **Regla del Boy Scout:** Deja el código un poco mejor de como lo encontraste.
- **Deuda Técnica:** Si por velocidad tomaste un atajo, documéntalo como "TODO: Refactor" y asígnale un tiempo en el próximo ciclo de mantenimiento.

---

## 3. Feedback del Cliente

El software debe servir al negocio.
- **Métricas de Uso:** Analiza qué módulos se usan más y cuáles fallan.
- **Entrevistas:** Pregunta al cliente: "¿Qué tarea te quita más tiempo hoy?". Esa es tu próxima feature pagada.

---

## 4. Evolución del Stack

El mundo tecnológico cambia.
- **Revisión Trimestral:** ¿Salió una versión LTS de Node o Java? ¿Hay una nueva librería que ahorra 50% de código?
- **Actualización de este Cerebro (Guía de Desarrollo de Software con IA):** Si descubres un flujo mejor, actualiza estos documentos. Este repositorio es un ente vivo.

---

## 5. Modernización de Sistemas Legacy (Migración)

Esta guía sirve para arreglar o migrar proyectos existentes. Sigue este orden de ataque:

### Fase A: El Diagnóstico
1. **Audit de Seguridad:** Usa `docs/07-SEGURIDAD.md` y `docs/29-MANTENIMIENTO.md` para cerrar brechas críticas.
2. **Logging de Emergencia:** Implementa `docs/13-LOGGING.md`. No intentes arreglar lo que no puedes ver.
3. **Mapeo de Deuda:** Usa `docs/01-PRINCIPIOS.md` para identificar qué patrones (SOLID/KISS) se están rompiendo.

### Fase B: Estabilización
1. **Replicación Local:** Usa `docs/25-DEBUGGING.md` para traer el sistema a un entorno controlado.
2. **Backups:** Antes de tocar nada, valida tu estrategia con `docs/15-BACKUPS.md`.
3. **Containerización:** Mete el sistema en Docker siguiendo `docs/11-DESPLIEGUE.md` para eliminar el "en mi máquina funciona".

### Fase C: Evolución
No reescribas todo de golpe. Usa el **Patrón Strangler**:
1. Extrae funcionalidades críticas a nuevos módulos siguiendo `docs/04-ESTRUCTURA.md`.
2. Conecta con el sistema viejo mediante adaptadores (`docs/01-PRINCIPIOS.md#patrones-de-diseno`).
3. Migra la infraestructura a los estándares de `docs/12-INFRAESTRUCTURA.md`.
