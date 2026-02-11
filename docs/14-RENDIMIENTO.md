# 14 - Rendimiento y Optimización

> Un sitio lento pierde usuarios y dinero. Optimizar no es opcional.

---

## Metricas Objetivo (Core Web Vitals)

| METRICA                    | BUENO      | NECESITA MEJORA | MALO |
|---------------------------|------------|-----------------|--------|
| **LCP** (Visualización)   | < 2.5s     | 2.5 - 4s        | > 4s |
| **INP** (Interactividad)  | < 200ms    | 200 - 500ms     | > 500ms |
| **CLS** (Estabilidad)     | < 0.1      | 0.1 - 0.25      | > 0.25 |

Herramientas de medicion: Lighthouse, PageSpeed Insights.

---

## Frontend (Angular)

### Imagenes
- Formato **WebP** o **AVIF** (30-50% mas ligero).
- `NgOptimizedImage` (`ngSrc`): Usa la directiva nativa de Angular para lazy loading y preconnect automático.
- Dimensiones explicitas (width/height) para evitar CLS.

### JavaScript (Angular CLI)
- **Code splitting:** Lazy load de rutas (`loadComponent`, `loadChildren`).
- **Tree shaking:** Activado por defecto en `ng build --configuration production`.
- **Deferrables (@defer):** Carga componentes no críticos de forma diferida.

```html
@defer (on viewport) {
  <heavy-chart-component />
} @placeholder {
  <div>Cargando gráfico...</div>
}
```

### Detección de Cambios (Change Detection)
- **Signal-based components:** Usar `changeDetection: ChangeDetectionStrategy.OnPush` SIEMPRE.
- Evitar funciones en templates `{{ calculateTotal() }}`. Usar **Signals** (`computed`).

### Renderizado
- **SSR / Hydration:** Usar `@angular/ssr` para SEO y First Paint rápido.
- **Virtual Scrolling:** `cdk-virtual-scroll-viewport` para listas largas (> 50 items).

---

## Backend (NestJS)

### Queries a Base de Datos
- Indices en columnas de `WHERE`, `JOIN`, `ORDER BY`.
- Evitar N+1: usar eager loading (`relations: ['user']`).
- Paginar resultados (`take: 50`, `skip: 0`).
- No traer `SELECT *` en produccion. Solo columnas necesarias.

### Caching
| CAPA | HERRAMIENTA | DURACION TIPICA |
|------|-------------|------------------|
| **HTTP** | Cache-Control | 1h - 1 semana |
| **App** | Redis (`cache-manager`) | 5min - 1h |
| **BD** | Materialized Views | Depende del caso |

### API
- Comprimir respuestas (gzip/brotli) con `compression` middleware.
- Connection pooling para BD.
- Rate limiting para proteger y controlar carga (`ThrottlerModule`).

---

## Base de Datos

### Indices

```sql
-- Index basico en columna de busqueda frecuente
CREATE INDEX idx_productos_categoria ON productos(categoria_id);
```

### Migraciones
- Usar migraciones para TODOS los cambios de esquema.
- Nunca modificar BD manualmente en produccion.

---

## Checklist de Rendimiento Pre-Deploy

- [ ] `ng build` sin errores ni warnings de tamaño excesivo.
- [ ] Imágenes optimizadas (`ngSrc`).
- [ ] Lazy loading en rutas principales.
- [ ] Índices de BD creados.
- [ ] Gzip activo en Nginx/Backend.
