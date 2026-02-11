# 31 - Gestión de Estado Global

> El estado global mal gestionado es la principal fuente de bugs en aplicaciones frontend.

---

## Filosofía de Estado en Angular Moderno

### Pregunta Clave
Antes de crear un store global, pregúntate:

```
¿REALMENTE necesito estado global?

SI:  Datos compartidos por 3+ componentes no relacionados (auth, carrito, notificaciones)
NO:  Datos de una página específica (usar Signal local)
NO:  Datos del servidor (usar RXJS / TanStack Query)
```

---

## Tipos de Estado

### 1. Estado Local (Signals)
Vive dentro del componente. Se pierde al desmontar.

```typescript
@Component({...})
export class CounterComponent {
  count = signal(0);
  double = computed(() => this.count() * 2);

  increment() {
    this.count.update(v => v + 1);
  }
}
```

**Usar cuando:**
- Solo un componente necesita los datos.
- El estado se resetea al cambiar de página.

### 2. Estado Global (NgRx SignalStore)
Compartido entre múltiples componentes. Persiste en navegación.
Moderno, sin boilerplate, type-safe.

```typescript
// auth.store.ts
import { signalStore, withState, withMethods } from '@ngrx/signals';

export const AuthStore = signalStore(
  { providedIn: 'root' },
  withState({ user: null, token: null }),
  withMethods((store) => ({
    login(user, token) {
      patchState(store, { user, token });
    },
    logout() {
      patchState(store, { user: null, token: null });
    }
  }))
);
```

**Usar cuando:**
- Múltiples componentes no relacionados necesitan los datos.
- Datos que persisten entre navegaciones (auth, carrito, preferencias).

### 3. Estado del Servidor (Resource API / RXJS)
Datos que vienen del backend.

```typescript
// Angular 19/20+ Resource API (Experimental)
import { resource } from '@angular/core';

@Component({...})
export class ProductList {
  // Carga automática basada en signals
  products = resource({
    loader: () => fetch('/api/products').then(r => r.json())
  });

  // products.value() -> T | undefined
  // products.isLoading() -> boolean
  // products.error() -> unknown
}
```

```typescript
// RXJS Clásico (AsyncPipe)
products$ = this.productService.getProducts();
```

---

## Herramientas Recomendadas

| Herramienta | Caso de Uso | Nota |
|-------------|-------------|------|
| **Signals** (Nativo) | Estado local y comunicación padre-hijo | Estándar desde Angular 16+ |
| **NgRx SignalStore** | Estado global ligero | Recomendado por defecto |
| **NgRx Classic** | Estado global complejo (Enterprise) | Solo si necesitas Time Travel o Effects muy complejos |
| **TanStack Query** | Estado del servidor | Si la app tiene mucho caching/refetching |

---

## Anti-Patrones

### 1. "Prop Drilling" excesivo
Pasar `@Input` a través de 5 niveles de componentes.
**Solución:** Usar un Service con Signals o SignalStore.

### 2. Mutar estado directamente (en objetos)
```typescript
// MAL
this.user().name = 'Juan'; // Signal no se entera

// BIEN
this.user.update(u => ({ ...u, name: 'Juan' }));
```

### 3. Usar `BehaviorSubject` para todo
Aunque es válido, Signals es más eficiente y requiere menos limpieza (`unsubscribe`).

---

## Persistencia

Usa `localStorage` para persistir datos entre recargas (F5), pero sincronízalo con tu Store.

```typescript
// NgRx SignalStore con persistencia (plugin)
export const AuthStore = signalStore(
  withState(initialState),
  withStorageSync('auth') // Plugin custom o de librería
);
```
