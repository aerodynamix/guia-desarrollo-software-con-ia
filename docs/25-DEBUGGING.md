# 25 - DEBUGGING Y RESOLUCIÓN DE PROBLEMAS (PROTOCOLO AVANZADO)

> "No puedo arreglar lo que no puedo reproducir."  
> El debugging efectivo requiere: replicación local, logs estratégicos, y análisis sistemático.

---

## SECCION 1: FILOSOFÍA DE DEBUGGING

### 1.1 Principios Fundamentales

1. **NUNCA DEBUGUEES EN PRODUCCIÓN:**
   - Replica el entorno localmente
   - Usa datos anonimizados
   - Protege la experiencia del usuario

2. **METODOLOGÍA CIENTÍFICA:**
   - Hipótesis: ¿Qué puede estar causando el error?
   - Experimento: Reproduce el escenario exacto
   - Observación: Analiza logs y comportamiento
   - Conclusión: Identifica la causa raíz
   - Solución: Implementa y verifica el fix

3. **DOCUMENTACIÓN:**
   - Cada bug crítico debe tener un issue en Git
   - Documenta la causa raíz y la solución
   - Comparte el conocimiento con el equipo

---

## SECCION 2: REPLICACIÓN DE ENTORNO DE PRODUCCIÓN

### 2.1 Clonación de Base de Datos

#### A) PostgreSQL: Streaming Remoto a Local

**Comando Mágico (SSH + Docker Pipe):**
```bash
# Vuelca la BD remota y la inyecta directo en tu contenedor local
# VENTAJA: No crea archivos intermedios, es streaming directo
ssh usuario@servidor-produccion.com "pg_dump -U postgres -h localhost mi_base_prod" \
  | docker exec -i contenedor-postgres-local psql -U postgres mi_base_local

# Verificar que se importó correctamente
docker exec -it contenedor-postgres-local psql -U postgres mi_base_local -c "\dt"
```

**Variante: Si NO usas Docker**
```bash
ssh usuario@servidor "pg_dump -U postgres mi_base_prod" | psql -U postgres mi_base_local
```

#### B) MySQL: Streaming Remoto a Local

```bash
ssh usuario@servidor "mysqldump -u root -p mi_base_prod" \
  | docker exec -i contenedor-mysql-local mysql -u root -ppassword mi_base_local
```

#### C) MongoDB: Exportar e Importar

```bash
# En el servidor remoto
ssh usuario@servidor "mongodump --db mi_base_prod --archive" > dump.archive

# En tu máquina local
docker exec -i contenedor-mongo-local mongorestore --archive < dump.archive
```

### 2.2 Anonimización de Datos (GDPR/LOPD)

**CRÍTICO: NUNCA trabajes con datos reales de usuarios en local**

**Script SQL de Anonimización (PostgreSQL):**
```sql
-- Ejecutar INMEDIATAMENTE después de clonar la BD

BEGIN;

-- Anonimizar usuarios
UPDATE users SET
  email = CONCAT('user_', id, '@example.test'),
  password_hash = '$2b$10$fakehashinvalidforlogin',
  phone = '+000000000',
  first_name = 'Usuario',
  last_name = CONCAT('Anon', id),
  address = 'Dirección Ficticia ' || id,
  city = 'Santiago',
  country = 'Chile';

-- Anonimizar métodos de pago
UPDATE payment_methods SET
  card_last_four = '0000',
  cardholder_name = 'TEST USER',
  card_brand = 'visa',
  expiry_date = '12/99';

-- Anonimizar órdenes sensibles
UPDATE orders SET
  shipping_address = 'Dirección de Envío Ficticia',
  billing_address = 'Dirección de Facturación Ficticia';

-- Limpiar logs de acceso
TRUNCATE TABLE audit_logs;
TRUNCATE TABLE login_attempts;

-- Resetear tokens de sesión
UPDATE sessions SET token = md5(random()::text);

COMMIT;

-- Verificar anonimización
SELECT email, phone, first_name FROM users LIMIT 5;
```

**Script para MongoDB:**
```javascript
// Conectar a la BD local
use mi_base_local

// Anonimizar usuarios
db.users.updateMany(
  {},
  {
    $set: {
      email: { $concat: ["user_", "$_id", "@example.test"] },
      password: "$2b$10$fakehashinvalid",
      phone: "+000000000",
      firstName: "Usuario",
      lastName: "Anon"
    }
  }
)

// Limpiar sesiones
db.sessions.drop()
db.auditLogs.drop()
```

### 2.3 Clonación de Variables de Entorno

**Paso 1: Exportar .env de Producción (Sin Secretos)**
```bash
# En el servidor de producción
ssh usuario@servidor "cat /app/.env" > .env.production

# Editar manualmente para REEMPLAZAR:
# - DATABASE_URL con URL local
# - API Keys de producción con API Keys de desarrollo/sandbox
# - SMTP credentials con servicio de testing (Mailtrap, Ethereal)
```

**Paso 2: Crear .env.local**
```bash
# .env.local (para debugging)
NODE_ENV=development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/mi_base_local

# API Keys de SANDBOX (NUNCA usar las de producción)
STRIPE_SECRET_KEY=sk_test_XXXXXXXXXX
SENDGRID_API_KEY=SG.test_XXXXXXXXXX

# Servicios de testing
SMTP_HOST=smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=test_user
SMTP_PASS=test_pass

# Logs más verbosos
LOG_LEVEL=debug
```

---

## SECCION 3: ANÁLISIS DE LOGS

### 3.1 Logs de Aplicación (Node.js/NestJS)

**Ver logs en tiempo real:**
```bash
# Docker
docker logs -f --tail=200 nombre-contenedor

# Filtrar por nivel de error
docker logs nombre-contenedor 2>&1 | grep -i "error\|exception\|fatal"

# Ver solo logs de las últimas 5 minutos
docker logs --since 5m nombre-contenedor

# Guardar logs a archivo para análisis
docker logs nombre-contenedor > debug_$(date +%Y%m%d_%H%M%S).log
```

**Logs estructurados (JSON) para análisis:**
```typescript
// logger.service.ts (NestJS)
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class LoggerService {
  private logger = new Logger('AppLogger');

  error(message: string, trace?: string, context?: any) {
    this.logger.error({
      message,
      trace,
      context,
      timestamp: new Date().toISOString(),
      environment: process.env.NODE_ENV
    });
  }

  debug(message: string, data?: any) {
    if (process.env.LOG_LEVEL === 'debug') {
      this.logger.debug({
        message,
        data,
        timestamp: new Date().toISOString()
      });
    }
  }
}
```

### 3.2 Logs de Base de Datos (PostgreSQL)

**Habilitar logging de queries lentas:**
```sql
-- Ejecutar en PostgreSQL
ALTER SYSTEM SET log_min_duration_statement = 1000; -- 1 segundo
SELECT pg_reload_conf();

-- Ver logs
docker exec -it contenedor-postgres tail -f /var/lib/postgresql/data/log/postgresql-*.log
```

**Analizar queries más lentas:**
```sql
-- Extensión pg_stat_statements (si está habilitada)
SELECT
  query,
  calls,
  total_time,
  mean_time,
  max_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

### 3.3 Logs de Servidor Web (Nginx/Apache)

**Nginx:**
```bash
# Access logs (quién accedió)
docker exec -it nginx tail -f /var/log/nginx/access.log

# Error logs (errores 4xx, 5xx)
docker exec -it nginx tail -f /var/log/nginx/error.log

# Filtrar solo errores 500
docker exec -it nginx tail -f /var/log/nginx/error.log | grep "500"
```

---

## SECCION 4: DEBUGGING DE CONTENEDORES

### 4.1 Inspección de Estado

**Ver estado del contenedor:**
```bash
docker ps -a  # Listar todos los contenedores (incluso detenidos)

docker inspect nombre-contenedor | jq '.[0].State'
# Salida:
# {
#   "Status": "running",
#   "Running": true,
#   "Paused": false,
#   "Restarting": false,
#   "ExitCode": 0,
#   "Error": "",
#   "StartedAt": "2026-02-09T10:30:00Z",
#   "FinishedAt": "0001-01-01T00:00:00Z"
# }
```

**Verificar reinicios automáticos:**
```bash
docker inspect nombre-contenedor | jq '.[0].RestartCount'

# Si es > 0, el contenedor se está reiniciando constantemente
# Ver por qué:
docker logs --tail 50 nombre-contenedor
```

### 4.2 Recursos del Contenedor (CPU/RAM)

**Monitoreo en tiempo real:**
```bash
docker stats nombre-contenedor

# Salida:
# CONTAINER ID   NAME        CPU %     MEM USAGE / LIMIT     NET I/O
# abc123         mi-app      35.2%     512MB / 2GB           1.2kB / 850B
```

**Si el contenedor usa demasiada RAM:**
```bash
# Entrar al contenedor y analizar procesos
docker exec -it mi-app top

# Ver qué proceso consume más memoria
docker exec -it mi-app ps aux --sort=-%mem | head -10
```

### 4.3 Entrar al Contenedor (Shell Interactiva)

```bash
# Si el contenedor tiene bash
docker exec -it nombre-contenedor /bin/bash

# Si es Alpine Linux (solo sh)
docker exec -it nombre-contenedor /bin/sh

# Una vez dentro:
ls -la /app                  # Verificar archivos
cat /app/.env                # Ver variables de entorno
env                          # Ver todas las variables
ps aux                       # Procesos corriendo
df -h                        # Espacio en disco
cat /app/logs/error.log      # Leer logs internos
```

---

## SECCION 5: DEBUGGING DE RED

### 5.1 Verificar Puertos Abiertos

**Desde tu máquina local:**
```bash
# Verifica si el puerto está escuchando
netstat -tuln | grep 3000

# Prueba conexión TCP
telnet localhost 3000

# Si telnet no está instalado, usa curl
curl -v http://localhost:3000

# Verificar qué proceso está usando el puerto
lsof -i :3000  # Linux/macOS
```

**Desde dentro del contenedor:**
```bash
docker exec -it mi-app netstat -tuln

# Verificar conectividad a la base de datos
docker exec -it mi-app ping db
docker exec -it mi-app curl -v http://db:5432
```

### 5.2 Debugging de DNS en Docker

**Problema común: "db: Name or service not known"**

```bash
# Verificar que los contenedores están en la misma red
docker network ls
docker network inspect mi-red | jq '.[0].Containers'

# Conectar contenedor a la red (si no está)
docker network connect mi-red nombre-contenedor

# Verificar resolución DNS desde el contenedor
docker exec -it mi-app nslookup db
docker exec -it mi-app ping db -c 3
```

### 5.3 Debugging de CORS

**Si el frontend no puede conectar al backend:**

```javascript
// Backend (NestJS): main.ts
app.enableCors({
  origin: process.env.FRONTEND_URL || 'http://localhost:5173',
  credentials: true
});

// Frontend: Verificar en DevTools > Network
// Buscar header:
// Access-Control-Allow-Origin: http://localhost:5173
```

---

## SECCION 6: ANÁLISIS DE IMÁGENES ADJUNTAS

Cuando el supervisor envía una captura de pantalla del error:

### 6.1 Protocolo de Análisis Visual

**Checklist del Agente IA:**

1. **Identificar el tipo de error:**
   - [ ] Error de consola del navegador (F12)
   - [ ] Mensaje de error en UI
   - [ ] Pantalla en blanco (White Screen of Death)
   - [ ] Error de red (Network tab)
   - [ ] Datos incorrectos en tabla/formulario

2. **Extraer información clave:**
   - **Texto del error:** Transcribir exactamente el mensaje
   - **Stack trace:** Identificar la línea de código que falla
   - **Network request:** ¿Qué endpoint está fallando? (Status code)
   - **Estado de la aplicación:** ¿Qué acción provocó el error?

3. **Análisis de consola del navegador:**
   ```
   EJEMPLO DE IMAGEN:
   
   Console (F12):
   ❌ TypeError: Cannot read property 'id' of undefined
       at ProductCard.vue:42
       at Array.map (native)
   
   ANÁLISIS DEL AGENTE:
   - Error en línea 42 de ProductCard.vue
   - Está intentando acceder a `item.id` pero `item` es undefined
   - Esto ocurre dentro de un .map(), lo que sugiere que el array no tiene datos
   - Hipótesis: El fetch de productos está fallando o retornando null
   ```

4. **Análisis de Network tab:**
   ```
   EJEMPLO DE IMAGEN:
   
   Network tab muestra:
   GET /api/products -> Status 500 Internal Server Error
   
   ANÁLISIS DEL AGENTE:
   - El endpoint /api/products está fallando en el backend
   - Necesito ver los logs del servidor para saber por qué
   - Posibles causas: query SQL incorrecta, BD caída, error de lógica
   ```

### 6.2 Ejemplo de Interacción con Imagen

```
SUPERVISOR: [Adjunta imagen de error]

AGENTE IA:
"Veo que tienes un error 500 en el endpoint GET /api/products. 
El mensaje de error en el frontend es 'TypeError: Cannot read property id of undefined'.

Necesito verificar los logs del backend. ¿Puedes ejecutar este comando y compartirme la salida?

docker logs --tail 50 nombre-contenedor-backend

Mientras tanto, voy a revisar el código de ProductCard.vue línea 42 para ver cómo está manejando el caso cuando products está vacío."
```

---

## SECCION 7: TÉCNICAS AVANZADAS DE DEBUGGING

### 7.1 Debugging de Performance (Slow Queries)

**Identificar queries lentas en PostgreSQL:**
```sql
-- Habilitar logging de queries lentas
ALTER SYSTEM SET log_min_duration_statement = 500; -- 500ms
SELECT pg_reload_conf();

-- Ver queries en ejecución
SELECT
  pid,
  now() - query_start as duration,
  query
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 seconds';
```

**Analizar plan de ejecución:**
```sql
EXPLAIN ANALYZE
SELECT p.*, c.name as category_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.active = true
ORDER BY p.created_at DESC
LIMIT 20;

-- Si "Seq Scan" aparece, falta un índice
-- Si "cost" es muy alto, la query necesita optimización
```

### 7.2 Memory Leaks (Fugas de Memoria)

**Detectar fugas en Node.js:**
```bash
# Generar heap snapshot
docker exec -it mi-app node --inspect=0.0.0.0:9229 dist/main.js

# Conectar Chrome DevTools:
# chrome://inspect
# Tomar snapshots en diferentes momentos y comparar
```

**Prevención:**
```typescript
// MAL: EventListener sin cleanup
class UserService {
  constructor(private eventEmitter: EventEmitter) {
    this.eventEmitter.on('user.created', this.handleUserCreated);
  }
  
  handleUserCreated(user) {
    // ...
  }
}

// BIEN: Cleanup en destroy
class UserService implements OnDestroy {
  constructor(private eventEmitter: EventEmitter) {
    this.eventEmitter.on('user.created', this.handleUserCreated);
  }
  
  onDestroy() {
    this.eventEmitter.off('user.created', this.handleUserCreated);
  }
}
```

### 7.3 Race Conditions

**Detectar:**
```typescript
// Síntoma: Datos inconsistentes en BD
// Ejemplo: Inventario negativo

// MAL (Race condition)
async decrementStock(productId: string, quantity: number) {
  const product = await this.findById(productId);
  product.stock -= quantity;
  await this.save(product);
  // Si 2 usuarios compran al mismo tiempo, ambos leen stock=10
  // y ambos escriben stock=9, cuando debería ser stock=8
}

// BIEN (Transacción atómica)
async decrementStock(productId: string, quantity: number) {
  await this.db.query(
    'UPDATE products SET stock = stock - $1 WHERE id = $2',
    [quantity, productId]
  );
}
```

---

## SECCION 8: HERRAMIENTAS DE DEBUGGING

### 8.1 Debugger Integrado en VSCode

**Configuración launch.json:**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Attach to Docker",
      "port": 9229,
      "address": "localhost",
      "localRoot": "${workspaceFolder}/src",
      "remoteRoot": "/app/src",
      "protocol": "inspector",
      "restart": true
    }
  ]
}
```

**Modificar docker-compose.yml:**
```yaml
services:
  app:
    command: node --inspect=0.0.0.0:9229 dist/main.js
    ports:
      - "3000:3000"
      - "9229:9229"  # Puerto de debugging
```

### 8.2 Postman/Bruno para Debugging de APIs

**Guardar colección de pruebas:**
```json
// bruno/api-tests/products/get-all.bru
meta {
  name: Get All Products
  type: http
}

get {
  url: {{baseUrl}}/api/products
  headers {
    Authorization: Bearer {{token}}
  }
}

tests {
  test("Status es 200", function() {
    expect(res.status).to.equal(200);
  });
  
  test("Retorna array de productos", function() {
    expect(res.body).to.be.an('array');
    expect(res.body.length).to.be.greaterThan(0);
  });
}
```

---

## SECCION 9: PROTOCOLO DE RESOLUCIÓN PARA EL AGENTE IA

### 9.1 Checklist Obligatorio

Cuando el supervisor reporte un bug:

1. **Recopilación de Información:**
   - [ ] ¿Qué acción provocó el error?
   - [ ] ¿Es reproducible? (Siempre / A veces / Una sola vez)
   - [ ] ¿En qué entorno? (Producción / Staging / Local)
   - [ ] ¿Hay logs disponibles?
   - [ ] ¿Hay capturas de pantalla?

2. **Replicación Local:**
   - [ ] Clonar BD de producción (anonimizada)
   - [ ] Reproducir el escenario exacto
   - [ ] Confirmar que el error ocurre localmente

3. **Diagnóstico:**
   - [ ] Analizar logs (aplicación, BD, servidor web)
   - [ ] Identificar la causa raíz (no el síntoma)
   - [ ] Proponer solución

4. **Implementación del Fix:**
   - [ ] Escribir el código de corrección
   - [ ] Agregar test unitario que cubra el caso
   - [ ] Verificar que NO rompe funcionalidad existente
   - [ ] Commit con formato: `fix(módulo): descripción (#issue)`

5. **Validación:**
   - [ ] Probar localmente
   - [ ] Deploy a staging
   - [ ] Validar con el supervisor
   - [ ] Deploy a producción

### 9.2 Template de Reporte de Bug

**Issue en GitHub/GitLab:**
```markdown
## Bug: [Título descriptivo]

**Severidad:** P0 (Crítico) / P1 (Alto) / P2 (Medio) / P3 (Bajo)

**Descripción:**
Al intentar [acción], ocurre [error] en lugar de [comportamiento esperado].

**Pasos para Reproducir:**
1. Ir a [URL]
2. Hacer clic en [botón]
3. Ingresar [datos]
4. Observar error

**Comportamiento Esperado:**
Debería [comportamiento correcto].

**Comportamiento Actual:**
Muestra error: [mensaje de error exacto]

**Logs:**
```
[Pegar logs aquí]
```

**Captura de Pantalla:**
[Adjuntar imagen]

**Entorno:**
- OS: [Windows/Linux/macOS]
- Navegador: [Chrome 122 / Firefox 123]
- Versión de la app: [v1.2.3]
```

---

## RESUMEN PARA EL AGENTE IA

**Cuando el supervisor reporte un bug:**

1. Solicita información completa (logs, imágenes, pasos)
2. Replica localmente (clona BD anonimizada si es necesario)
3. Analiza logs sistemáticamente
4. Identifica causa raíz (no el síntoma)
5. Propón solución con código
6. Genera test que cubra el caso
7. Valida antes de deploy a producción

**Herramientas clave:**
- `docker logs` para logs de contenedores
- `docker exec -it` para shell interactiva
- Bruno/Postman para testing de APIs
- VSCode Debugger para debugging paso a paso
- PostgreSQL `EXPLAIN ANALYZE` para optimizar queries

**NUNCA:**
- Debuguees directo en producción
- Uses datos reales de usuarios en local
- Adivines la solución sin analizar logs

---

**FIN DE DOCUMENTO DE DEBUGGING**

Este protocolo asegura resolución efectiva de bugs sin impactar al usuario final.
