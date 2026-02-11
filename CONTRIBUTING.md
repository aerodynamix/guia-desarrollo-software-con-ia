# GUÍA DE CONTRIBUCIÓN

¡Gracias por tu interés en contribuir a la "Guía de Desarrollo de Software con IA"! Este proyecto busca ser el estándar de facto para el desarrollo de software moderno asistido por IA.

---

## CÓDIGO DE CONDUCTA

Este proyecto se rige por un Código de Conducta simple: **Sé profesional y respetuoso.**

- **Cero discriminación:** No toleramos acoso ni discriminación de ningún tipo.
- **Comunicación técnica:** Mantén las discusiones centradas en la tecnología y la arquitectura.
- **Crítica constructiva:** Ataca las ideas, no a las personas.

---

## ¿CÓMO CONTRIBUIR?

### 1. Reportar Errores

Si encuentras un error en la documentación o código de ejemplo, por favor abre un Issue con:
- Título descriptivo.
- Pasos para reproducir (si aplica).
- Solución propuesta (opcional pero recomendada).

### 2. Proponer Mejoras (RFC)

Para cambios significativos (nuevas tecnologías, cambios de arquitectura, reestructuración):

1. **Abre un Issue** con la etiqueta `RFC` (Request for Comments).
2. **Describe el problema** actual.
3. **Propón tu solución** detallada.
4. **Espera feedback** de la comunidad y mantenedores antes de codificar.

### 3. Enviar Cambios (Pull Requests)

1. **Haz un Fork** del repositorio.
2. **Crea una Rama** para tu feature o fix: `git checkout -b feature/nueva-seccion-seguridad` o `git checkout -b fix/typo-en-readme`.
3. **Realiza tus cambios** siguiendo los estándares del proyecto.
4. **Haz Commit** usando [Conventional Commits](https://www.conventionalcommits.org/):
   - `docs: agregar sección sobre OWASP`
   - `fix: corregir enlace roto en README`
   - `style: formatear tablas en STACK.md`
5. **Abre un Pull Request (PR)** hacia la rama `main` del repositorio original.

---

## ESTÁNDARES DE DOCUMENTACIÓN

- **Markdown:** Usa GitHub Flavored Markdown.
- **Idioma:** Español neutro y técnico.
- **Ortografía:** Revisa tildes y gramática.
- **Sin Emojis:** No uses emojis en la documentación técnica (salvo excepciones muy justificadas en secciones "humanas" como esta guía).
- **Concisión:** Ve al grano. Menos es más.

---

## PREGUNTAS FRECUENTES

**¿Puedo proponer un nuevo framework para el stack?**
Sí, pero debes justificarlo muy bien en un RFC. El stack actual (Angular/NestJS/PostgreSQL) es la base "opinada" del proyecto. Solo agregaremos alternativas si cubren casos de uso no resueltos.

**¿Cómo ejecuto el proyecto localmente?**
Este es un repositorio de documentación. Puedes usar cualquier editor de Markdown (VS Code recomendado) o servirlo con `npx serve` si fuera necesario, pero generalmente basta con la vista previa del editor.
