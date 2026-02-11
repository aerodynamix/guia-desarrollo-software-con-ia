# 08 - Testing y Garantía de Calidad

> Codigo sin tests es codigo que no se puede cambiar con confianza.
> API sin documentacion es API que nadie usa.

---

## Testing

### Piramide de Tests

```
            /  E2E  \          Pocos, lentos, costosos
           /----------\
          / Integracion \      Moderados
         /----------------\
        /    Unitarios      \  Muchos, rapidos, baratos
       /____________________\
```

### Tipos de Test por Proyecto

| PROYECTO         | UNITARIOS | INTEGRACION | E2E |
|------------------|-----------|-------------|-----|
| Landing Page     | No        | No          | Opcional (Lighthouse) |
| Ecommerce        | Si (Core) | Si          | Si (Checkout) |
| Dashboard        | Si (Utils)| Si          | Opcional |
| SaaS             | **Obligatorio** | **Obligatorio** | **Obligatorio** |
| API              | Si        | **Obligatorio** | No (usa Bruno/Postman) |

### Herramientas Recomendadas

| Tipo | Herramienta | Contexto |
|------|-------------|----------|
| **Unitarios Angular** | **Jest / Vitest** | Componentes, Services, Pipes. (Karma es legacy) |
| **Unitarios NestJS** | **Jest** | Services, Utils, Guards. (Default de NestJS) |
| **Integracion API** | **Supertest** | Endpoints de NestJS (HTTP Requests reales) |
| **E2E Web** | **Playwright** | Flujos completos en navegador real (Safari, Chrome, Firefox) |
| **Mocks** | **Jest / Mockito** | Simular dependencias externas (Pagos, Email) |

---

## Testing en Angular (Frontend)

Preferimos **Angular Testing Library** sobre `TestBed` puro para tests más robustos y centrados en el usuario.

### Ejemplo Unitario (Componente)

```typescript
import { render, screen, fireEvent } from '@testing-library/angular';
import { CounterComponent } from './counter.component';

test('debe incrementar el contador al hacer click', async () => {
  await render(CounterComponent);

  const button = screen.getByRole('button', { name: /incrementar/i });
  const count = screen.getByTestId('count-display');

  expect(count.textContent).toBe('0');
  fireEvent.click(button);
  expect(count.textContent).toBe('1');
});
```

### Ejemplo Unitario (Service)

```typescript
// auth.service.spec.ts
describe('AuthService', () => {
  let service: AuthService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [AuthService],
      imports: [HttpClientTestingModule]
    });
    service = TestBed.inject(AuthService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it('login debe retornar usuario y token', () => {
    service.login('test@test.com', '123456').subscribe(res => {
      expect(res.token).toBe('fake-jwt-token');
    });

    const req = httpMock.expectOne('/api/auth/login');
    expect(req.request.method).toBe('POST');
    req.flush({ token: 'fake-jwt-token', user: { id: 1 } });
  });
});
```

---

## Testing en NestJS (Backend)

### Ejemplo Unitario (Service)
```typescript
describe('ProductsService', () => {
  let service: ProductsService;
  let repo: Repository<Product>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        ProductsService,
        {
          provide: getRepositoryToken(Product),
          useValue: { find: jest.fn().mockResolvedValue([]) }, // Mock del Repository
        },
      ],
    }).compile();

    service = module.get(ProductsService);
    repo = module.get(getRepositoryToken(Product));
  });

  it('findAll debe retornar array de productos', async () => {
    const result = await service.findAll();
    expect(result).toEqual([]);
    expect(repo.find).toHaveBeenCalled();
  });
});
```

### Ejemplo Integración (E2E API)
```typescript
describe('AuthModule (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    await app.init();
  });

  it('/auth/login (POST) - Credenciales validas', () => {
    return request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'admin@test.com', password: 'password123' })
      .expect(201)
      .expect((res) => {
        expect(res.body.accessToken).toBeDefined();
      });
  });
});
```

---

## Documentacion de API (Swagger/OpenAPI)

### Configuracion NestJS

```typescript
// main.ts
const config = new DocumentBuilder()
  .setTitle('Mi API')
  .setDescription('Documentacion de la API')
  .setVersion('1.0')
  .addBearerAuth()
  .build();
const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

### Decoradores
Usa decoradores para documentar DTOs y Controllers:
```typescript
@ApiProperty({ example: 'juan@gmail.com', description: 'Correo del usuario' })
email: string;

@ApiOperation({ summary: 'Crear usuario' })
@ApiResponse({ status: 201, description: 'Usuario creado exitosamente.' })
create(@Body() createUserDto: CreateUserDto) { ... }
```

---

## Prueba Manual de APIs: Bruno

Preferimos **Bruno** sobre Postman porque los archivos `.bru` se guardan en el repositorio git.

### Estructura en el Repo
```
/bruno
  /auth
    login.bru
    register.bru
  /products
    list.bru
  environments/
    local.bru
    prod.bru
  bruno.json
```

**Ventaja:** Todo el equipo tiene las colecciones de prueba actualizadas al hacer `git pull`.

---

## Simulación de Errores (Sad Path)

No testes solo el "camino feliz".
*   **Jest:** `jest.spyOn(service, 'pagar').mockRejectedValue(new Error('Pago fallido'))`

**Objetivo:** Verificar que el sistema captura el error y responde `400 Bad Request` o `500 Internal Error` sin caerse.
