# 06 - Git (Flujo de Versionado)

> El historial de Git es la bitácora del proyecto. Cuida su legibilidad.

---

## 1. Estrategia de Ramas (Escalable)

No compliques el flujo si el equipo es pequeño (1-3 personas).

### Nivel 1: Flujo Simple (Proyectos Pequeños/Medianos)
Ideal para MVPs o equipos ágiles.
*   **main**: Producción (Estable).
*   **develop**: Desarrollo/Integración (Beta).
*   **Flujo**: Se trabaja directo en `develop` (o en ramas cortas `feat/`). Se fusiona `develop` -> `main` para desplegar.

### Nivel 2: Git Flow Completo (Proyectos Corporativos)
Para equipos grandes con ciclos de QA estrictos.
*   **main**: Producción.
*   **develop**: Integración.
*   **feature/nombre-feature**: Nuevas funcionalidades (salen de develop).
*   **release/v1.0**: Preparación para release (congelado de código).
*   **hotfix/bug-critico**: Parches urgentes a producción.

### Resumen de Ramas
| Rama | Nivel | Despliega a | Uso |
|------|-------|-------------|-----|
| **main** | 1 & 2 | **Prod** | Versión pública final. |
| **develop** | 1 & 2 | **Dev/Staging** | Trabajo en curso. |
| **feature/*** | 2 | - | Tarea específica aislada. |

---

## 2. Cheat Sheet: Comandos Git (CLI)

### Actualizar tu rama (Evitar conflictos)
```bash
git checkout develop
git pull origin develop
git checkout feature/mi-rama
git merge develop
```

### Fusionar Feature (Al terminar)
```bash
git checkout develop
git merge --no-ff feature/mi-rama
# --no-ff preserva la historia de que existió una rama
git push origin develop
```

### Resolución de Conflictos (Paso a Paso)
1.  Status: `git status` (Muestra archivos rojos).
2.  Abrir archivo con conflicto.
3.  Buscar `<<<<<<< HEAD` (Lo tuyo) y `>>>>>>>` (Lo que llega).
4.  Elegir código correcto y borrar marcas.
5.  Git Add: `git add archivo_corregido.js`
6.  Finalizar: `git commit -m "fix: resolver conflicto en merge"`

---

## 3. Protocolo de Commits con IA (Español OBLIGATORIO)

La IA debe sugerir el comando de commit en **ESPAÑOL**.

### Opción A: Conventional Commits (Estándar)
`tipo(ámbito): descripción breve (#IssueID)`

**Tipos:**
*   `feat`: Nueva funcionalidad.
*   `fix`: Corrección de bug.
*   `docs`: Documentación.
*   `style`: Formato.
*   `refactor`: Cambio de código sin funcionalidad nueva.
*   `test`: Tests.
*   `chore`: Mantenimiento.

**Ejemplos:**
*   `feat(auth): implementar login con google (#12)`
*   `fix(carrito): corregir cálculo de iva en checkout (#45)`

### Opción B: Custom / Legacy
Si el humano prefiere un formato específico (ej: Jira).
`PREFIJO: #ID Descripción`

**Ejemplos:**
*   `CHG: #123 Implementación de login`
*   `BUG: #456 Error en suma de totales`

---

## 4. Gestión de Issues y PRs

Nada se codifica sin un Issue (Tarea) asociado.

### Vinculación (Smart Commits)
Usa palabras clave en tus commits para actualizar issues automáticamente:
*   `Closes #123`: Cierra el issue al mergear a main.
*   `Fixes #45`: Arregla un bug referenciado.
*   `Relates to #10`: Menciona el issue sin cerrarlo.

### Checklist de PR
*   [ ] Título descriptivo (Conventional Commit).
*   [ ] Descripción de "Qué hace" y "Por qué".
*   [ ] Tests pasan (CI en verde).
*   [ ] Self-review del código antes de pedir review a otro.

---

## 5. Comandos Avanzados (Corrección)

Usa estos comandos con precaución. Nunca hagas rebase/reset en ramas públicas compartidas.

### Corregir el último commit (Sin cambiar mensaje)
Si olvidaste agregar un archivo:
1. `git add archivo_olvidado.js`
2. `git commit --amend --no-edit`

### Limpiar historial local (Interactive Rebase)
Si tienes 3 commits basura ("wip", "typo", "fix") y quieres unirlos en uno solo antes de hacer push:
1. `git rebase -i HEAD~3`
2. Cambia `pick` por `squash` (o `s`) en los commits que quieres fusionar hacia arriba.
3. Termina de fusionar y haz force push **SOLO SI** no has compartido la rama.

### Revertir cambios
*   **Mal commit (Local):** `git reset --soft HEAD~1` (Te devuelve los archivos al staging area).
*   **Mal commit (Publicado/Push):** `git revert <commit-hash>` (Crea un NUEVO commit que invierte los cambios, historial seguro).
