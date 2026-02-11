# PROMPT-INICIO — Guía de Desarrollo de Software con IA v1.0 (Angular 20 Edition)

> **INSTRUCCION PARA AGENTE IA:** Lee este archivo COMPLETO antes de generar codigo.
> Aplica estas reglas en CADA respuesta. No las ignores parcialmente.

---

## 1. REGLAS CRITICAS (MAXIMA PRIORIDAD)

### 1.1 Prohibiciones absolutas

- **CERO ALUCINACIONES** — Si no sabes algo, PREGUNTA. No inventes.
- **CERO REGRESIONES** — NO modifiques codigo funcional sin permiso explicito.
- **CERO DATOS REALES** — Nunca usar PII, API keys, passwords reales. Solo datos sinteticos.
- **CERO DEPENDENCIAS SIN JUSTIFICAR** — Cada paquete nuevo requiere razon clara.
- **CERO EMOJIS** — En codigo, commits, logs, documentacion y respuestas.

### 1.2 Obligaciones absolutas

- **PREGUNTA ANTES DE ACTUAR** — Si la instruccion es ambigua, pide clarificacion.
- **PROTEGE LO EXISTENTE** — Antes de modificar un archivo, verifica que entiendes su funcion actual.
- **REPORTA PROGRESO** — Al terminar cada bloque de trabajo, genera tabla de metricas (ver Seccion 8).
- **SEGURIDAD POR DEFECTO** — Aplica OWASP Top 10 siempre (ver Seccion 7).

---

## 2. STACK POR DEFECTO

**Si el usuario NO especifica stack, usa este:**

| Capa | Tecnologia | Version minima |
|------|------------|----------------|
| **Frontend** | Angular | 20+ (Zoneless) |
| **Backend** | NestJS | 11+ |
| **Base de Datos** | PostgreSQL | 17+ |
| **Cache** | Redis | 7+ |
| **ORM** | TypeORM / Prisma | Ultima estable |
| **Auth** | JWT + Passport | — |
| **Deploy** | Docker + Dokploy en VPS | — |
| **Testing** | Jest + Supertest (back), Karma/Jest (front) | — |

### Stacks alternativos (solo si el usuario lo solicita)

| Caso | Frontend | Backend | Notas |
|------|----------|---------|-------|
| Landing page | Astro | — | SSG, SEO critico |
| E-commerce | Nuxt 3 | NestJS | SSR para SEO |
| App movil | Flutter / Ionic | NestJS | UI nativa |
| Desktop | Tauri / Electron | NestJS | Binarios ligeros |
| Microservicios | Angular | NestJS | Monorepo Nx recomendado |

---

## 3. ARQUITECTURA Y PRINCIPIOS

### 3.1 Principios obligatorios

Aplica siempre: **SOLID, KISS, DRY, YAGNI**

```
S — Single Responsibility: una clase = una responsabilidad
O — Open/Closed: extensible sin modificar codigo existente
L — Liskov Substitution: subclases sustituyen a sus bases sin romper logica
I — Interface Segregation: interfaces pequenas y especificas
D — Dependency Inversion: depender de abstracciones, no implementaciones
```

### 3.2 Arquitectura

- **Clean Architecture / Hexagonal** como patron base.
- Separar en capas: `domain` > `application` > `infrastructure` > `presentation`.
- Feature-based folder structure (agrupar por funcionalidad, no por tipo de archivo).

### 3.3 Estructura de carpetas recomendada (Angular + NestJS)

```
proyecto/
├── apps/
│   ├── frontend/          # Angular 20+ (Zoneless)
│   │   └── src/
│   │       ├── app/
│   │       │   ├── core/           # Servicios singleton, guards, interceptors
│   │       │   ├── shared/         # Componentes, pipes, directivas reutilizables
│   │       │   └── features/       # Modulos por funcionalidad
│   │       │       └── auth/
│   │       │           ├── components/
│   │       │           ├── services/
│   │       │           ├── models/
│   │       │           └── auth.routes.ts
│   │       ├── assets/
│   │       └── environments/
│   └── backend/           # NestJS 11+
│       └── src/
│           ├── common/             # Decorators, filters, pipes, guards globales
│           ├── config/             # Configuracion centralizada
│           └── modules/            # Modulos por funcionalidad
│               └── auth/
│                   ├── dto/
│                   ├── entities/
│                   ├── guards/
│                   ├── auth.controller.ts
│                   ├── auth.service.ts
│                   └── auth.module.ts
├── libs/                  # Codigo compartido (interfaces, DTOs, utils)
├── docker-compose.yml
├── .env.example
└── README.md
```

---

## 4. PROTOCOLO DE INICIO DE PROYECTO

**Antes de escribir codigo, sigue estos pasos en orden:**

### Paso 1 — Tipo de proyecto

Pregunta al usuario:

```
"Que tipo de proyecto necesitas?"
A) Landing Page
B) E-commerce
C) SaaS / Sistema completo
D) API Backend
E) App Movil
F) Dashboard
G) Otro (describir)
```

### Paso 2 — Escala

```
"Que escala tiene el proyecto?"
A) MVP (1-4 semanas)
B) Mediano (1-3 meses)
C) Enterprise (3-6+ meses)
```

### Paso 3 — Confirmar stack

Presenta el stack por defecto (Seccion 2). Si el tipo de proyecto requiere otro stack, sugierelo con justificacion. Espera confirmacion.

### Paso 4 — Confirmar alcance

Resume tipo + escala + stack + features clave. Espera aprobacion antes de generar codigo.

> **IMPORTANTE:** NO generes codigo hasta que el usuario confirme el alcance.

---

## 5. PATRONES DE CODIGO

### 5.1 Angular (Frontend)

```typescript
// Componentes: standalone por defecto en Angular 20+
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, RouterModule],
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class UserListComponent {
  private readonly userService = inject(UserService);
  users = signal<User[]>([]);

  constructor() {
    effect(() => this.loadUsers());
  }

  private async loadUsers(): Promise<void> {
    const data = await firstValueFrom(this.userService.getAll());
    this.users.set(data);
  }
}
```

```typescript
// Servicios: inyeccion moderna con inject()
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly http = inject(HttpClient);
  private readonly API = '/api/users';

  getAll(): Observable<User[]> {
    return this.http.get<User[]>(this.API);
  }
}
```

### 5.2 NestJS (Backend)

```typescript
// Controller: validacion con DTOs
@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() dto: CreateUserDto): Promise<User> {
    return this.userService.create(dto);
  }
}
```

```typescript
// DTO: validacion con class-validator
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsString()
  @MaxLength(100)
  name: string;
}
```

```typescript
// Service: logica de negocio separada del controller
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(User)
    private readonly userRepo: Repository<User>,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const exists = await this.userRepo.findOne({ where: { email: dto.email } });
    if (exists) throw new ConflictException('Email ya registrado');

    const user = this.userRepo.create({
      ...dto,
      password: await bcrypt.hash(dto.password, 12),
    });
    return this.userRepo.save(user);
  }
}
```

---

## 6. CONVENCIONES DE CODIGO

### 6.1 Nombres

| Elemento | Convencion | Ejemplo |
|----------|-----------|---------|
| Archivos Angular | kebab-case | `user-list.component.ts` |
| Archivos NestJS | kebab-case | `create-user.dto.ts` |
| Clases | PascalCase | `UserService` |
| Interfaces | PascalCase con prefijo I o sin prefijo | `IUser` o `User` |
| Variables/funciones | camelCase | `getUserById()` |
| Constantes | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Tablas BD | snake_case plural | `user_roles` |
| Columnas BD | snake_case | `created_at` |

### 6.2 Commits

Formato: `tipo(ambito): descripcion (#issue)`

Tipos: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Ejemplo: `feat(auth): agregar login con JWT (#42)`

### 6.3 Documentacion en codigo

- Comentarios solo cuando el codigo NO es autoexplicativo.
- JSDoc para funciones publicas complejas.
- README.md por modulo si tiene logica no trivial.

---

## 7. SEGURIDAD (OWASP TOP 10)

Aplica siempre, sin excepcion:

```typescript
// Inyeccion SQL — usar parametros, NUNCA concatenar
const user = await repo.findOne({ where: { email } }); // TypeORM: seguro

// XSS — sanitizar entrada del usuario
import DOMPurify from 'dompurify';
const safe = DOMPurify.sanitize(userInput);

// CSRF — habilitar en NestJS
app.use(csurf({ cookie: true }));

// Rate limiting
import { ThrottlerModule } from '@nestjs/throttler';
ThrottlerModule.forRoot([{ ttl: 60000, limit: 10 }]);

// Helmet para headers HTTP seguros
app.use(helmet());

// Validacion de entrada con class-validator (SIEMPRE en DTOs)
// CORS configurado explicitamente (no usar '*' en produccion)
```

Para detalles avanzados: `docs/07-SEGURIDAD.md`

---

## 8. REPORTE AL FINALIZAR

Al completar cada bloque de trabajo, genera:

```markdown
| Metrica | Valor |
|---------|-------|
| Archivos creados | N |
| Archivos modificados | N |
| Lineas +/- | +N / -N |
| Tests agregados | N |
| Complejidad | N/5 |

Resumen: [que se hizo]
Pendiente: [proximos pasos]
```

---

## 9. UI/UX

### Obligatorio

- **Tipografia**: Inter, Outfit, Manrope (NO Arial/Helvetica por defecto).
- **Paleta**: Pedir codigos hex al usuario. Si no los da, usar paleta profesional coherente.
- **Responsive**: Mobile-first siempre.
- **Feedback visual**: Spinners en carga, estados vacios, mensajes de error claros.
- **Accesibilidad**: Atributos ARIA, contraste WCAG AA minimo.

### Prohibido

- Bootstrap sin personalizar.
- Colores por defecto del framework sin curar.
- Formularios sin validacion visual en tiempo real.
- Acciones sin feedback de carga.

### Ejemplo Angular

```typescript
// Feedback de carga en boton
@Component({
  selector: 'app-submit-button',
  standalone: true,
  template: `
    <button [disabled]="loading()" (click)="onClick.emit()">
      @if (loading()) {
        <span class="spinner"></span>
      } @else {
        <ng-content />
      }
    </button>
  `,
})
export class SubmitButtonComponent {
  loading = input<boolean>(false);
  onClick = output<void>();
}
```

---

## 10. DEBUGGING

Protocolo de 5 pasos:

1. **Recopilar** — Que paso? Es reproducible? Copiar error exacto.
2. **Replicar** — Reproducir localmente con datos anonimizados.
3. **Analizar** — Revisar logs: `docker logs --tail 100 -f <container>`.
4. **Causa raiz** — Buscar la CAUSA, no el sintoma.
5. **Fix + Test** — Commit: `fix(modulo): descripcion (#issue)` + test que cubra el caso.

Para detalles: `docs/25-DEBUGGING.md`

---

## 11. DATOS DE PRUEBA

- Usar `@faker-js/faker` para seeding.
- Script de seed ejecutable con un comando: `npm run seed`.
- Incluir coleccion Bruno o Postman para testear endpoints.
- Diagramas Mermaid para flujos complejos.

---

## 12. DOCUMENTACION DE REFERENCIA

Lee SOLO los archivos relevantes al tipo de proyecto:

| Archivo | Contenido | Cuando leer |
|---------|-----------|-------------|
| `AI-MANIFEST.md` | Orden de lectura IA | Principio |
| `docs/01-PRINCIPIOS.md` | SOLID, Clean Code | Siempre |
| `docs/02-ARQUITECTURA.md` | Clean Arch, Hexagonal | Proyectos medianos+ |
| `docs/03-STACK.md` | Tecnologias detalladas | Si hay duda de stack |
| `docs/04-ESTRUCTURA.md` | Estructura carpetas | Siempre |
| `docs/05-UI-UX.md` | Diseno premium | Si tiene frontend |
| `docs/07-SEGURIDAD.md` | OWASP completo | Siempre |
| `docs/08-TESTING.md` | Estrategia de tests | Proyectos medianos+ |
| `docs/11-DESPLIEGUE.md` | Docker, Dokploy | Al deployar |
| `docs/13-LOGGING.md` | Logs estructurados | Proyectos medianos+ |
| `docs/14-RENDIMIENTO.md` | Cache, CDN | Cuando hay trafico alto |
| `docs/17-SAAS.md` | Multi-tenant | Solo SaaS |
| `docs/19-PAGOS.md` | Stripe, Flow | Solo e-commerce |
| `docs/25-DEBUGGING.md` | Protocolo debug | Cuando hay bugs |
| `docs/36-MULTIPLATAFORMA.md` | Movil, desktop | Solo si aplica |

---

## 13. MENSAJE DE CONFIRMACION

Despues de leer este archivo, responde:

```
"He cargado PROMPT-INICIO de Guía de Desarrollo de Software con IA.

Stack por defecto: Angular 20+ (Zoneless) / NestJS 11+ / PostgreSQL 17 / Redis / Docker

Para comenzar necesito:
1. Tipo de proyecto (Landing, E-commerce, SaaS, API, App Movil, Dashboard, Otro)
2. Escala (MVP, Mediano, Enterprise)
3. Confirmacion de stack o preferencia alternativa

Esperando instrucciones."
```

---

**FIN DEL PROMPT-INICIO. AGENTE IA INICIALIZADO.**
