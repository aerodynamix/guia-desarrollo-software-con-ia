# 40 - Guía de Migración y Modernización

> **Objetivo:** Transformar proyectos legados en aplicaciones modernas, mantenibles y escalables usando el stack Angular 20+ / NestJS 11.

---

## Estrategia General: "The Strangler Fig Pattern"

Nunca reescribas todo de cero ("Big Bang"). Migra incrementalmente.

1.  **Identificar:** Elige un módulo pequeño y aislado (ej. "Perfil de Usuario").
2.  **Estrangular:** Crea el nuevo módulo en el nuevo stack y enruta el tráfico hacia él.
3.  **Eliminar:** Borra el código antiguo cuando el nuevo esté estable.
4.  **Repetir.**

---

## Frontend: Migrando a Angular 20+ (Zoneless)

### 1. De Vue (Options/Composition API) a Angular Signals

| Vue (Legacy) | Angular 20+ (Modern) | Notas |
|---|---|---|
| `data() { return { count: 0 } }` | `count = signal(0)` | Reactividad fina sin Zone.js |
| `computed: { double() { ... } }` | `double = computed(() => ...)` | Derivación pura |
| `watch(count, (val) => ...)` | `effect(() => ...)` | Efectos secundarios controlados |
| `props: ['title']` | `title = input<string>()` | Signal-based inputs |
| `emits: ['save']` | `save = output<void>()` | Output moderno |
| `v-if` / `v-for` | `@if` / `@for` | Control flow blocks (más performante) |
| `Slot` | `ng-content` | Proyección de contenido |

**Ejemplo Práctico:**

*Vue (Legacy)*
```javascript
export default {
  data() { return { count: 0 } },
  methods: { inc() { this.count++ } }
}
```

*Angular (Modern)*
```typescript
@Component({
  template: `<button (click)="inc()">{{ count() }}</button>`
})
export class Counter {
  count = signal(0);
  inc() { this.count.update(c => c + 1); }
}
```

### 2. De React (Hooks) a Angular Signals

| React | Angular 20+ | Notas |
|---|---|---|
| `useState(0)` | `signal(0)` | Sin reglas de hooks complejas |
| `useEffect(() => ...)` | `effect(() => ...)` | Se ejecuta automáticamente al cambiar señales |
| `useContext` | `inject(MyToken)` | Inyección de dependencias superior |
| `useMemo` | `computed` | Memorización automática |

### 3. De Angular Legacy (Modules/Zone.js) a Standalone/Zoneless

1.  **Eliminar NgModules:** Convertir todos los componentes a `standalone: true`.
2.  **Inyección:** Usar `inject()` en lugar de constructores.
3.  **Change Detection:**
    - Activar `provideExperimentalZonelessChangeDetection()` en `app.config.ts`.
    - Eliminar `zone.js` de `polyfills.ts`.
    - Usar `ChangeDetectorRef.markForCheck()` es innecesario con Signals.
4.  **Control Flow:** Migrar `*ngIf` a `@if` (usar `ng g @angular/core:control-flow`).

---

## Backend: Migrando a NestJS 11

### 1. De Express (Raw) a NestJS

| Express | NestJS | Razón del Cambio |
|---|---|---|
| `app.get('/user', (req, res) => ...)` | `@Get('user') getUser() { ... }` | Decoradores son más declarativos |
| `middleware` | `Middleware` / `Guard` / `Interceptor` | Separación de responsabilidades |
| `req.body` | DTO (`@Body() dto: CreateUserDto`) | Validación automática con `class-validator` |
| `mongoose.connect(...)` | `MongooseModule.forRoot(...)` | Inyección de dependencias |
| `winston` setup manual | `Logger` integrado | Logging estandarizado |

### 2. Patrón de Arquitectura (Clean Architecture)

Al migrar, no copies la estructura "spaghetti". Refactoriza a capas:

1.  **Controllers:** Solo reciben HTTP y llaman al servicio. NO lógica de negocio.
2.  **Services:** Lógica de negocio pura.
3.  **Repositories:** Acceso a datos (TypeORM/Prisma).

---

## Plan de Refactorización paso a paso

### Fase 1: Preparación
- [ ] Configurar Monorepo (Nx o Turborepo) para alojar código nuevo y viejo juntos.
- [ ] Configurar linters estrictos (ESLint + Prettier).
- [ ] Instalar Angular 20+ y NestJS 11.

### Fase 2: Backend (API Gateway)
- [ ] Crear un "Proxy" en NestJS que redirija al backend viejo.
- [ ] Migrar endpoint por endpoint (empezando por los más usados o criticos).
- [ ] Validar DTOs estrictamente en el nuevo backend.

### Fase 3: Frontend (Micro-frontends o Componentes)
- [ ] Migrar componentes UI base (Botones, Inputs) a Angular Material.
- [ ] Reemplazar lógica de estado global (Vuex/Redux) por **NgRx SignalStore**.
- [ ] Migrar vistas completas una por una.

---

## Anti-Patrones de Migración

1.  **"One-to-One Translation":** Intentar escribir código React en Angular.
    *   *Corrección:* Piensa en "Angular Way" (Inyección de dependencias, Servicios, Signals).
2.  **Ignorar TypeScript:** Usar `any` para migrar rápido.
    *   *Corrección:* Definir interfaces estrictas desde el día 1.
3.  **Mantener Zone.js:** Por miedo a romper cosas.
    *   *Corrección:* Zoneless es el futuro. Úsalo para evitar deuda técnica futura.
