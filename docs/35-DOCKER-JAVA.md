# 35 - Docker para Java y Spring Boot

> Docker para Java no es igual que para Node.js. Esta guía te enseña las optimizaciones específicas para JVM.

---

## Índice

1. [Dockerfile Multi-stage para Java](#1-dockerfile-multi-stage-para-java)
2. [Maven vs Gradle](#2-maven-vs-gradle)
3. [Optimizaciones JVM](#3-optimizaciones-jvm)
4. [Docker Compose para Spring Boot](#4-docker-compose-para-spring-boot)
5. [Configuración Spring para Docker](#5-configuración-spring-para-docker)
6. [Troubleshooting Java en Docker](#6-troubleshooting-java-en-docker)

---

## 1. Dockerfile Multi-stage para Java

### Maven (Spring Boot)

```dockerfile
# ===== BUILDER STAGE (Compilación) =====
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

# Copiar archivos de configuración Maven primero (para cache de layers)
COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .

# Descargar dependencias (se cachea si pom.xml no cambia)
RUN ./mvnw dependency:go-offline -B

# Copiar código fuente
COPY src ./src

# Compilar aplicación (skip tests para build más rápido)
RUN ./mvnw clean package -DskipTests -B

# ===== RUNNER STAGE (Ejecución) =====
FROM eclipse-temurin:21-jre-alpine

# Crear usuario no-root (SEGURIDAD CRÍTICA)
RUN addgroup -g 1001 spring && \
    adduser -S spring -u 1001 -G spring

WORKDIR /app

# Copiar solo el JAR compilado desde builder
COPY --from=builder --chown=spring:spring /app/target/*.jar app.jar

# Cambiar a usuario no-root
USER spring

# Exponer puerto
EXPOSE 8080

# Health check (requiere Spring Boot Actuator)
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Variables de entorno para JVM
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+UseContainerSupport"

# Comando de inicio
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**Tamaño resultante:**
- Imagen con JDK completo: ~450MB
- Imagen con JRE: ~150MB
- Imagen con JRE personalizado (jlink): ~80MB

### Gradle (Spring Boot)

```dockerfile
# ===== BUILDER STAGE =====
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

# Copiar archivos Gradle
COPY build.gradle settings.gradle gradlew ./
COPY gradle ./gradle

# Descargar dependencias
RUN ./gradlew dependencies --no-daemon

# Copiar código fuente
COPY src ./src

# Compilar (bootJar crea el executable jar)
RUN ./gradlew bootJar --no-daemon

# ===== RUNNER STAGE =====
FROM eclipse-temurin:21-jre-alpine

RUN addgroup -g 1001 spring && \
    adduser -S spring -u 1001 -G spring

WORKDIR /app

# Copiar JAR desde build/libs/
COPY --from=builder --chown=spring:spring /app/build/libs/*.jar app.jar

USER spring
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC -XX:+UseContainerSupport"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

## 2. Maven vs Gradle

### Comparativa para Docker

```
ASPECTO              | MAVEN                    | GRADLE
---------------------|--------------------------|-------------------------
Tiempo de build      | Más lento (~2-3 min)     | Más rápido (~1-2 min)
Cache de deps        | dependency:go-offline    | gradle dependencies
Configuración        | pom.xml (verboso)        | build.gradle (conciso)
Imagen final         | Igual (~150MB)           | Igual (~150MB)
Ecosistema Spring    | Más estándar             | Más moderno
Recomendación        | Enterprise/Legacy        | Nuevos proyectos
```

**Decisión:**
- Usa **Maven** si tu equipo ya lo conoce o proyecto existente
- Usa **Gradle** para proyectos nuevos (builds más rápidos)

---

## 3. Optimizaciones JVM

### Flags Esenciales para Contenedores

```bash
# MÍNIMO (obligatorio)
JAVA_OPTS="-XX:+UseContainerSupport"

# RECOMENDADO (producción)
JAVA_OPTS="
  -Xms256m 
  -Xmx512m 
  -XX:+UseG1GC 
  -XX:MaxGCPauseMillis=100 
  -XX:+UseContainerSupport
  -XX:InitialRAMPercentage=50.0
  -XX:MaxRAMPercentage=80.0
"

# AVANZADO (alta carga)
JAVA_OPTS="
  -Xms512m 
  -Xmx1024m 
  -XX:+UseG1GC 
  -XX:MaxGCPauseMillis=100 
  -XX:+UseContainerSupport
  -XX:InitialRAMPercentage=50.0
  -XX:MaxRAMPercentage=80.0
  -XX:+HeapDumpOnOutOfMemoryError
  -XX:HeapDumpPath=/app/logs/heap-dump.hprof
  -XX:+PrintGCDetails
  -XX:+PrintGCDateStamps
  -Xloggc:/app/logs/gc.log
"
```

### Explicación de Flags

```
FLAG                          | QUÉ HACE
------------------------------|------------------------------------------
-Xms256m                      | Heap inicial (mínimo)
-Xmx512m                      | Heap máximo
-XX:+UseG1GC                  | Usa G1 Garbage Collector (moderno)
-XX:MaxGCPauseMillis=100      | Pausas GC máx 100ms (low latency)
-XX:+UseContainerSupport      | CRÍTICO: Respeta límites de container
-XX:InitialRAMPercentage=50   | Usa 50% RAM disponible al inicio
-XX:MaxRAMPercentage=80       | Nunca excede 80% RAM del container
-XX:+HeapDumpOnOutOfMemoryError| Genera dump si OOM (debugging)
```

### Sizing de Memoria

```
USUARIOS      | HEAP MIN | HEAP MAX | RAM CONTAINER
--------------|----------|----------|---------------
<1k (dev)     | 128m     | 256m     | 512m
1k-10k        | 256m     | 512m     | 1g
10k-50k       | 512m     | 1024m    | 2g
50k-100k      | 1g       | 2g       | 4g
100k+         | 2g       | 4g       | 8g
```

**Regla de Oro:** RAM del container = Heap Max * 1.5 (para non-heap, threads, metaspace)

### JRE Personalizado con JLink (Opcional)

Para imágenes ULTRA pequeñas (~80MB):

```dockerfile
# STAGE 1: Crear JRE mínimo
FROM eclipse-temurin:21-jdk-alpine AS jre-builder

RUN jlink \
    --add-modules java.base,java.logging,java.xml,java.sql,java.naming,java.management,java.instrument,jdk.unsupported \
    --strip-debug \
    --no-man-pages \
    --no-header-files \
    --compress=2 \
    --output /jre-minimal

# STAGE 2: Builder (igual que antes)
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
# ... resto igual

# STAGE 3: Runner con JRE personalizado
FROM alpine:latest

# Copiar JRE mínimo
COPY --from=jre-builder /jre-minimal /opt/jre
ENV PATH="/opt/jre/bin:$PATH"

RUN addgroup -g 1001 spring && \
    adduser -S spring -u 1001 -G spring

WORKDIR /app
COPY --from=builder --chown=spring:spring /app/target/*.jar app.jar

USER spring
EXPOSE 8080

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

## 4. Docker Compose para Spring Boot

### Stack Completo (API + PostgreSQL + Redis)

```yaml
version: '3.8'

services:
  # Spring Boot API
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: springboot-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      # Spring profiles
      SPRING_PROFILES_ACTIVE: production
      
      # Base de datos
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/miapp
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD}
      
      # Pool de conexiones
      SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE: 10
      SPRING_DATASOURCE_HIKARI_MINIMUM_IDLE: 2
      
      # JPA/Hibernate
      SPRING_JPA_HIBERNATE_DDL_AUTO: validate
      SPRING_JPA_SHOW_SQL: false
      SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT: org.hibernate.dialect.PostgreSQLDialect
      
      # Redis (cache/sessions)
      SPRING_DATA_REDIS_HOST: redis
      SPRING_DATA_REDIS_PORT: 6379
      
      # Actuator
      MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE: health,info,metrics,prometheus
      MANAGEMENT_ENDPOINT_HEALTH_SHOW_DETAILS: when_authorized
      
      # JVM Options
      JAVA_OPTS: -Xms512m -Xmx1024m -XX:+UseG1GC -XX:+UseContainerSupport
      
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    volumes:
      - ./logs:/app/logs
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

  # PostgreSQL
  db:
    image: postgres:17-alpine
    container_name: springboot-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: miapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=en_US.UTF-8"
    volumes:
      - postgres-data:/var/lib/postgresql/data
      # Scripts SQL iniciales o Flyway migrations
      - ./db/init:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"  # Exponer solo en dev
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    container_name: springboot-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"  # Exponer solo en dev
    networks:
      - app-network

  # Nginx (opcional - reverse proxy)
  nginx:
    image: nginx:alpine
    container_name: springboot-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - api
    networks:
      - app-network

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local

networks:
  app-network:
    driver: bridge
```

### Comandos Útiles

```bash
# Desarrollo (rebuild automático)
docker-compose up --build

# Producción (daemon)
docker-compose -f docker-compose.prod.yml up -d

# Ver logs de API
docker-compose logs -f api

# Ver estadísticas de memoria/CPU
docker stats springboot-api

# Ejecutar bash dentro del container
docker-compose exec api sh

# Ejecutar comando Maven/Gradle dentro del container
docker-compose exec api ./mvnw clean test

# Ver logs de base de datos
docker-compose logs -f db

# Backup de PostgreSQL
docker-compose exec db pg_dump -U postgres miapp > backup.sql

# Restore de PostgreSQL
docker-compose exec -T db psql -U postgres miapp < backup.sql
```

---

## 5. Configuración Spring para Docker

### application.yml (Base)

```yaml
spring:
  application:
    name: miapp
  
  # Datasource (variables de entorno)
  datasource:
    url: ${SPRING_DATASOURCE_URL:jdbc:postgresql://localhost:5432/miapp}
    username: ${SPRING_DATASOURCE_USERNAME:postgres}
    password: ${SPRING_DATASOURCE_PASSWORD:postgres}
    driver-class-name: org.postgresql.Driver
    
    # HikariCP (pool de conexiones)
    hikari:
      maximum-pool-size: ${SPRING_DATASOURCE_HIKARI_MAXIMUM_POOL_SIZE:10}
      minimum-idle: ${SPRING_DATASOURCE_HIKARI_MINIMUM_IDLE:2}
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
    
  # JPA/Hibernate
  jpa:
    hibernate:
      ddl-auto: ${SPRING_JPA_HIBERNATE_DDL_AUTO:validate}
    show-sql: ${SPRING_JPA_SHOW_SQL:false}
    properties:
      hibernate:
        format_sql: true
        dialect: ${SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT:org.hibernate.dialect.PostgreSQLDialect}
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
  
  # Redis
  data:
    redis:
      host: ${SPRING_DATA_REDIS_HOST:localhost}
      port: ${SPRING_DATA_REDIS_PORT:6379}
      password: ${SPRING_DATA_REDIS_PASSWORD:}
      timeout: 60000

# Actuator (health checks)
management:
  endpoints:
    web:
      exposure:
        include: ${MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE:health,info}
      base-path: /actuator
  endpoint:
    health:
      show-details: ${MANAGEMENT_ENDPOINT_HEALTH_SHOW_DETAILS:never}
  metrics:
    export:
      prometheus:
        enabled: true

# Server
server:
  port: 8080
  shutdown: graceful
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
  
# Logging
logging:
  level:
    root: INFO
    com.miapp: ${LOG_LEVEL:DEBUG}
    org.hibernate.SQL: ${SQL_LOG_LEVEL:DEBUG}
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: ${LOG_FILE:/app/logs/application.log}
    max-size: 10MB
    max-history: 30
```

### application-production.yml

```yaml
spring:
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate  # NUNCA usar update/create en prod

logging:
  level:
    root: WARN
    com.miapp: INFO
    org.hibernate.SQL: WARN

management:
  endpoint:
    health:
      show-details: when_authorized
```

### Flyway (Migrations)

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  flyway:
    enabled: true
    baseline-on-migrate: true
    locations: classpath:db/migration
```

Estructura:
```
src/main/resources/
  db/
    migration/
      V1__initial_schema.sql
      V2__add_users_table.sql
      V3__add_products_table.sql
```

---

## 6. Troubleshooting Java en Docker

### Problema: OutOfMemoryError

```bash
# Ver uso de memoria
docker stats springboot-api

# Inspeccionar límites
docker inspect springboot-api | grep -i memory

# Aumentar heap
JAVA_OPTS="-Xms512m -Xmx1024m"

# O aumentar límite del container
docker run -m 2g myapp
```

### Problema: Container se queda sin responder

```bash
# Generar thread dump
docker exec springboot-api jstack 1 > thread-dump.txt

# Generar heap dump
docker exec springboot-api jmap -dump:format=b,file=/tmp/heap.bin 1

# Copiar heap dump a host
docker cp springboot-api:/tmp/heap.bin ./heap.bin

# Analizar con VisualVM o Eclipse MAT
```

### Problema: Conexiones a BD agotadas

```yaml
# Aumentar pool de conexiones
spring:
  datasource:
    hikari:
      maximum-pool-size: 20  # Era 10
      minimum-idle: 5        # Era 2
      
# Ver conexiones activas
SELECT count(*) FROM pg_stat_activity WHERE datname = 'miapp';
```

### Problema: Build muy lento

```dockerfile
# Optimizar cache de Maven
RUN --mount=type=cache,target=/root/.m2 \
    ./mvnw dependency:go-offline

# O usar Gradle cache
RUN --mount=type=cache,target=/root/.gradle \
    ./gradlew dependencies
```

### Problema: Timezone incorrecto

```dockerfile
# Agregar en Dockerfile
ENV TZ=America/Santiago
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/$TZ /etc/localtime && \
    echo $TZ > /etc/timezone
```

---

## 7. Checklist Docker para Java

```
DOCKERFILE:
[ ] Multi-stage build (builder + runner)
[ ] Usuario no-root
[ ] Health check configurado
[ ] JAVA_OPTS con UseContainerSupport
[ ] Heap sizing apropiado
[ ] Imagen base: eclipse-temurin (no openjdk)

DOCKER-COMPOSE:
[ ] Variables de entorno externalizadas
[ ] Secrets NO hardcodeados
[ ] Depends_on con health checks
[ ] Volúmenes para logs
[ ] Network aislada

SPRING BOOT:
[ ] Actuator habilitado (/actuator/health)
[ ] Datasource con variables de entorno
[ ] Profile 'production' configurado
[ ] Flyway para migrations (no ddl-auto)
[ ] HikariCP pool configurado

PRODUCCIÓN:
[ ] Resource limits (CPU/memoria)
[ ] Graceful shutdown habilitado
[ ] Logs en JSON (para parsing)
[ ] Monitoring (Prometheus/Grafana)
[ ] Backups automatizados de BD
```

---

## 8. Recursos Java + Docker

- **Spring Boot Docker Guide:** https://spring.io/guides/gs/spring-boot-docker/
- **Eclipse Temurin:** https://adoptium.net/
- **Testcontainers (testing):** https://www.testcontainers.org/
- **JLink Documentation:** https://docs.oracle.com/en/java/javase/21/docs/specs/man/jlink.html
