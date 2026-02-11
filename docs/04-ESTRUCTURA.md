# 04 - Estructura de Proyecto y Seeding

> "Un lugar para cada cosa, y cada cosa en su lugar."
> La arquitectura debe gritar la intención del sistema.

---

## Estructura Estándar (Monorepo o Backend/Frontend)

Esta estructura sigue **Clean Architecture** simplificada para mantenibilidad.

### Root
```
/proyecto
├── .github/              # Actions (CI/CD)
├── .vscode/              # Configuración de editor compartida
├── docs/                 # Documentación (Copiar esta carpeta bases-software)
├── src/                  # Código Fuente
├── tests/                # Tests E2E / Integración
├── tools/                # Scripts de mantenimiento (backups, seeds)
├── .env.example          # Plantilla de variables (SIN SECRETOS)
├── .cursorrules          # Reglas para IA
├── CHANGELOG.md          # Historial de cambios
├── docker-compose.yml    # Orquestación local
├── package.json          # Scripts de ejecución
└── README.md             # Punto de entrada
```

> **Importante:** Antes de empezar, configura tu editor y entorno según [03-STACK.md](03-STACK.md#entorno-de-desarrollo-local).

### Backend (NestJS / Node)
```
src/
├── config/               # Validación de .env (Zod/Joi)
├── modules/              # Módulos de Dominio (Auth, Users, Payments)
│   ├── auth/
│   │   ├── dto/          # Data Transfer Objects (Input/Output)
│   │   ├── entities/     # Entidades de Dominio
│   │   ├── guards/       # Seguridad (Roles, JWT)
│   │   ├── strategies/   # Estrategias (Local, Google, Apple)
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.module.ts
├── shared/               # Código compartido (Utils, Constants)
├── database/
│   ├── migrations/       # Historial de cambios de esquema
│   └── seeds/            # Scripts de población de datos (Seeders)
└── main.ts               # Entry point
```

### Frontend (Angular 19+)
```
src/
├── app/
│   ├── core/             # Singleton services (Auth, HTTP Interceptors)
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── services/
│   ├── features/         # Módulos funcionales (Lazy Loaded)
│   │   ├── dashboard/    # Feature Module
│   │   │   ├── components/
│   │   │   ├── pages/
│   │   │   └── dashboard.routes.ts
│   │   └── products/
│   ├── shared/           # UI Kit y utilidades reutilizables
│   │   ├── components/   # Botones, Cards, Modals
│   │   ├── directives/
│   │   └── pipes/
│   ├── layouts/          # Estructuras de página (MainLayout, AuthLayout)
│   ├── app.config.ts     # Configuración global (Providers)
│   ├── app.routes.ts     # Rutas principales
│   └── app.component.ts  # Componente raíz
├── assets/               # Imágenes, i18n, iconos
├── environments/         # Variables por entorno
└── styles/               # CSS global / Temas
```

---

## Estrategia de Seeding (Poblado de Datos)

El **Seeding** es crítico para desarrollar rápido. Nadie debe crear usuarios manualmente en la BD local.

### Reglas de Oro
1.  **Idempostencia:** Ejecutar el seed 10 veces no debe duplicar datos ni romper la BD. (Usar `upsert` o `deleteMany` antes).
2.  **Datos Realistas:** Usa librerías como `@faker-js/faker` para generar nombres, emails y textos creíbles. Nada de "asd asd".
3.  **Determinismo (Opcional):** Configura la "semilla" del randomizer si necesitas que los datos sean siempre iguales snapshot tras snapshot.

### Herramientas Recomendadas
*   **TypeScript/JS:** `Prisma Seed` o scripts custom con `ts-node`.
*   **Librería Fake:** `@faker-js/faker` (Estándar de la industria).

### Ejemplo de Seeder (Prisma + Faker)

Crear archivo `/src/database/seeds/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client';
import { faker } from '@faker-js/faker';

const prisma = new PrismaClient();

async function main() {
  console.log('>> Iniciando Seeding...');

  // 1. Limpiar BD (Orden inverso a dependencias)
  await prisma.order.deleteMany();
  await prisma.user.deleteMany();

  // 2. Crear Usuarios Admin (Fijos)
  const admin = await prisma.user.upsert({
    where: { email: 'admin@empresa.com' },
    update: {},
    create: {
      email: 'admin@empresa.com',
      name: 'Super Admin',
      role: 'ADMIN',
      password: '$argon2id$...' // Hash pre-calculado de "123456"
    },
  });

  // 3. Crear Usuarios Fake (Aleatorios)
  const usersData = Array.from({ length: 50 }).map(() => ({
    email: faker.internet.email(),
    name: faker.person.fullName(),
    role: 'USER',
    avatar: faker.image.avatar(),
    createdAt: faker.date.past(),
  }));

  await prisma.user.createMany({ data: usersData });

  console.log('[OK] Seeding completado: 50 usuarios generados.');
}

main()
  .catch((e) => console.error(e))
  .finally(async () => await prisma.$disconnect());
```

### Ejecución
En `package.json`:
```json
"scripts": {
  "seed": "ts-node src/database/seeds/seed.ts"
}
```

Comando: `npm run seed`

---

## Seeding en Java (Spring Boot)

En Spring Boot, el estándar es usar un `CommandLineRunner` o `ApplicationRunner` que se ejecute solo en el perfil de `dev` o `test`.

### Herramientas Recomendadas
- **Librería Fake:** `net.datafaker:datafaker` (Sucesora moderna de JavaFaker).
- **Control de Entorno:** `@Profile("dev")`.

### Ejemplo de Seeder (Spring Data JPA + Datafaker)

```java
@Component
@Profile("dev")
@RequiredArgsConstructor
public class DatabaseSeeder implements CommandLineRunner {

    private final UserRepository userRepository;
    private final Faker faker = new Faker();

    @Override
    public void run(String... args) {
        if (userRepository.count() > 0) return; // Regla de Idempotencia

        List<User> users = IntStream.range(0, 50).mapToObj(i -> {
            User user = new User();
            user.setName(faker.name().fullName());
            user.setEmail(faker.internet().emailAddress());
            user.setRole(Role.USER);
            return user;
        }).toList();

        userRepository.saveAll(users);
        System.out.println("[OK] Seeding Java: 50 usuarios generados.");
    }
}
```

---

## Comprobación de Entorno (Health Checks)

Además de la estructura, asegura que el repo tenga:
1.  `.nvmrc` o `.node-version`: Para asegurar la misma versión de Node.
2.  `.editorconfig`: Para indentación consistente.
3.  `.dockerignore` y `.gitignore`: Para no subir basura.
