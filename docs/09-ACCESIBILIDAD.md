# 09 - Accesibilidad (a11y)

> Una web accesible llega a mas usuarios y cumple estandares legales.
> No es opcional en proyectos profesionales.

---

## WCAG 2.1 - Nivel Minimo: AA

### 4 Principios

| PRINCIPIO | SIGNIFICADO |
|-----------|-------------|
| **Perceptible** | El contenido se puede ver, oir o tocar |
| **Operable** | Se puede navegar con teclado y asistentes |
| **Comprensible** | El contenido y la UI son predecibles |
| **Robusto** | Funciona con tecnologias asistivas |

---

## Herramientas Angular (CDK A11y)

Angular provee `@angular/cdk/a11y` para resolver problemas complejos de accesibilidad.

### 1. FocusTrap (Modales)
Atrapa el foco dentro de un elemento (ej: Modal) para que el usuario no navegue el fondo.

```html
<div cdkTrapFocus [cdkTrapFocusAutoCapture]="true">
  <h2>Confirmar Acción</h2>
  <button (click)="confirm()">Sí</button>
  <button (click)="close()">No</button>
</div>
```

### 2. LiveAnnouncer (Lectores de Pantalla)
Anuncia cambios dinámicos a usuarios ciegos (ej: "Guardado exitosamente").

```typescript
export class MyComponent {
  constructor(private announcer: LiveAnnouncer) {}

  save() {
    this.announcer.announce('Cambios guardados');
  }
}
```

### 3. ListKeyManager (Navegación Compleja)
Maneja la navegación con flechas (arriba/abajo) en listas, menús y grids.

---

## Checklist Practico

### HTML Semantico

```html
<!-- MAL: todo con divs -->
<div class="header" (click)="navigate()">Inicio</div>

<!-- BIEN: etiquetas semanticas + interactividad nativa -->
<header>
  <nav aria-label="Menu principal">
    <a routerLink="/">Inicio</a>
  </nav>
</header>
```

Usar siempre:
*   `<header>, <nav>, <main>, <section>, <article>, <aside>, <footer>`
*   `<h1>` a `<h6>` en orden jerarquico (no saltar niveles)
*   `<button>` para acciones, `<a>` para navegacion
*   `<label>` asociado a cada `<input>`

### Imagenes

```html
<!-- Imagen informativa: alt descriptivo -->
<img [src]="imgUrl" alt="Grafico de ventas mensuales 2026, tendencia creciente">

<!-- Imagen decorativa: alt vacio -->
<img src="ornamento.svg" alt="" aria-hidden="true">

<!-- Icono con significado (Material Symbols) -->
<mat-icon aria-hidden="false" aria-label="Cerrar modal">close</mat-icon>
```

### Formularios (Angular Material)

Angular Material ya implementa accesibilidad por defecto, pero debes verificar:

```html
<!-- SIEMPRE asociar label con input -->
<mat-form-field>
  <mat-label>Correo electronico</mat-label>
  <input matInput [formControl]="emailCtrl" required>
  @if (emailCtrl.invalid) {
    <mat-error>Ingrese un email valido</mat-error>
  }
</mat-form-field>
```

### Navegacion con Teclado

Reglas:
*   Todo elemento interactivo debe ser alcanzable con `Tab`.
*   El foco debe ser VISIBLE (usa `:focus-visible` en CSS).
*   Evitar `tabindex` positivos (rompen el flujo natural). Usar `0` o `-1`.

### Colores y Contraste

| REQUISITO | RATIO MINIMO (AA) |
|-----------|-------------------|
| Texto normal (< 18px) | 4.5:1 |
| Texto grande (>= 18px) | 3:1 |
| Elementos UI (iconos) | 3:1 |

Herramientas: **Contrast Checker**, **lighthouse**.

---

## ARIA (Atributos de Accesibilidad)

**REGLA DE ORO:** Si puedes usar HTML nativo, NO uses ARIA.
Un `<button>` es mejor que `<div role="button">`.

Atributos mas usados en Angular:
```html
<!-- Descripcion cuando no hay texto visible -->
<button [attr.aria-label]="'Editar ' + item.name">  <mat-icon>edit</mat-icon></button>

<!-- Estado de menus -->
<button [attr.aria-expanded]="isOpen">Menu</button>

<!-- Mensajes urgentes -->
<div role="alert" *ngIf="error">{{ error }}</div>
```

---

## Testing de Accesibilidad

1.  **Automático:** Lighthouse en Chrome DevTools.
2.  **Linting:** `eslint-plugin-angular-a11y`
3.  **Manual:** Navega TU aplicación usando SOLO el teclado (`Tab`, `Shift+Tab`, `Enter`, `Space`, `Esc`). Si te trabas, es un bug de accesibilidad.
