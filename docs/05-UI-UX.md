# 05 - Diseño UI/UX Premium

> Cero interfaces genéricas o aburridas.
> Cada proyecto debe ser una pieza única de ingeniería visual, profesional y fluida.

---

## Principio Fundamental

El diseno NO es decoracion. Es comunicacion. Cada elemento visual tiene un proposito.
Si no lo tiene, no debe existir.

**NO HACER:**
- Copiar layouts de templates gratuitos sin adaptarlos
- Usar colores aleatorios sin paleta definida
- Poner iconos por poner
- Dejar espaciados inconsistentes
- Usar fuentes del sistema sin intencion

**SI HACER:**
- Definir paleta de colores ANTES de codear
- Establecer escala tipografica consistente
- Usar espaciado con sistema (4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px)
- Disenar mobile-first
- Usar transiciones y animaciones sutiles (no excesivas)

---

## Sistema de Diseno Minimo

Antes de escribir un componente visual, define:

### 1. Paleta de Colores

```css
:root {
  /* Primarios */
  --color-primary: #...;
  --color-primary-hover: #...;
  --color-primary-light: #...;

  /* Secundarios */
  --color-secondary: #...;

  /* Neutros */
  --color-bg: #...;
  --color-surface: #...;
  --color-border: #...;
  --color-text: #...;
  --color-text-muted: #...;

  /* Semanticos */
  --color-success: #...;
  --color-warning: #...;
  --color-error: #...;
  --color-info: #...;
}
```

Herramientas para generar paletas: Coolors.co, Realtime Colors, Tailwind palette generator.

### 2. Motor de Estilos (Implementación)

No hardcodees colores hexadecimales en el código (ej: `bg-[#FF0000]`).
Usa **Variables** para permitir cambios globales.

#### Opción A: Angular Material (Recomendado para Enterprise)
Usa el sistema de temas de Material Design 3. Customiza la paleta en `styles.scss`.

```scss
@use '@angular/material' as mat;

$my-primary: mat.define-palette(mat.$indigo-palette);
$my-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);

$my-theme: mat.define-light-theme((
 color: (
   primary: $my-primary,
   accent: $my-accent,
 ),
));

@include mat.all-component-themes($my-theme);
```

#### Opción B: TailwindCSS (Recomendado para Custom UI)
Configura `tailwind.config.js` para usar nombres semánticos.

```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981',
      }
    }
  }
}
// Uso: class="bg-primary text-white"
```

### 3. Tipografia

```
Headings:    Inter / Poppins / Manrope (sans-serif modernas)
Body:        Inter / DM Sans / Nunito
Monospace:   JetBrains Mono / Fira Code (para codigo/datos)
```

### 4. Espaciado (Sistema de 4px)

```
xs:  4px    (0.25rem)
sm:  8px    (0.5rem)
md:  16px   (1rem)
lg:  24px   (1.5rem)
xl:  32px   (2rem)
2xl: 48px   (3rem)
3xl: 64px   (4rem)
```

---

## Componentes con Diseno Premium

### Botones (No usar el default de HTML)

```
Primario:     Fondo solido + color primario + hover con leve oscurecimiento + transicion 150ms
Secundario:   Borde + fondo transparente + hover con fondo suave
Ghost:        Sin borde ni fondo + hover con fondo sutil
Peligro:      Rojo para acciones destructivas
Deshabilitado: Opacidad 0.5 + cursor not-allowed
```

### Cards
```
- Fondo surface con borde sutil o sombra (no ambos)
- Padding interno consistente (16px o 24px)
- Border radius 8-12px
- Hover con elevacion sutil (si es clicable)
```

### Tablas
```
- Header con fondo diferenciado
- Filas alternas con color sutil
- Hover en fila con highlight
- Bordes solo horizontales (mas limpio)
- Pagination clara
- Responsive: scroll horizontal o cards en mobile
```

### Formularios
```
- Labels siempre visibles (no solo placeholder)
- Estados claros: default, focus, error, success
- Mensajes de error debajo del campo en rojo
- Focus ring visible para accesibilidad
- Inputs con altura consistente (40-44px)
```

---

## Iconografia

| Libreria | Estilo | Uso Recomendado | Enlace |
|----------|--------|-----------------|--------|
| **Material Symbols** | Google, oficial | **Proyectos Angular**. Default. | fonts.google.com/icons |
| Lucide Icons | Lineal, moderno | Dashboards modernos | lucide.dev |
| Phosphor Icons | Versatil, 6 pesos | Todo proyecto | phosphoricons.com |
| Heroicons | Lineal/solido | Proyectos Tailwind | heroicons.com |

**Regla:** Usa UNA sola libreria de iconos por proyecto. No mezcles.

---

## Animaciones y Transiciones (Angular Animations)

Usa `@angular/animations` para transiciones complejas o CSS puro para simples.

```typescript
// Transicion simple de entrada (fade in)
trigger('fadeIn', [
  state('void', style({ opacity: 0 })),
  transition(':enter', [
    animate(300, style({ opacity: 1 }))
  ])
])
```

**Preferir `transform` y `opacity`** para animaciones fluidas (GPU-accelerated).

---

## Responsividad

```
Mobile first. SIEMPRE.

Breakpoints (Tailwind defaults):
  sm:  640px   (movil grande)
  md:  768px   (tablet)
  lg:  1024px  (laptop)
  xl:  1280px  (desktop)
  2xl: 1536px  (pantalla grande)

Regla: Si no se ve bien en 375px de ancho (iPhone SE), no esta terminado.
```

### Patron de Responsividad para Tablas
```
Desktop:  Tabla completa con todas las columnas
Tablet:   Ocultar columnas secundarias
Mobile:   Convertir filas en cards apiladas
```

---

## Estrategia de Modos (Claro / Oscuro)

Pregunta al usuario si requiere soporte para ambos o uno específico.

### Opción A: Ambos (Dark Mode Support)
Implementa un interruptor (toggle) que persista en `localStorage`.
- **Claro:** Colores vibrantes, contrastes nítidos.
- **Oscuro:** Gris muy oscuro (no negro puro #000), acentos legibles.

### Opción B: Modo Único Premium
Si el proyecto tiene una identidad fuerte (ej: un dashboard financiero oscuro o una landing médica clara), optimiza todos los recursos para ese modo único.

---

## Footer y Créditos (Marca Personal)

El pie de página no es basura. Es tu firma.

### Estructura Mínima
1.  **Copyright:** `© 2026 [Nombre Cliente]. Todos los derechos reservados.`
2.  **Enlaces Legales:** Privacidad, Términos. (Obligatorio por ley en muchos países).
3.  **Firma del Desarrollador (Discreta):**
    *   *Texto:* "Desarrollado con [Tecnología] por [Tu Nombre/Agencia]."
    *   *Estilo:* Texto pequeño (`text-xs`), color muted (`text-gray-400`), sin robar protagonismo.
    *   *Link:* Al portafolio o LinkedIn.
