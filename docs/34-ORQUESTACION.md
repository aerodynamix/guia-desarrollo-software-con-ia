# 34 - Orquestación de Contenedores (Docker, Kubernetes, Portainer)

> **"Contenedores sin orquestación = caos. Orquestación mal hecha = pesadilla."**
> Esta guía te enseña cuándo usar cada herramienta y cómo escalar sin perder la cordura.

---

## Índice Rápido

1. [Fundamentos de Contenedores](#1-fundamentos-de-contenedores)
2. [Docker y Docker Compose](#2-docker-y-docker-compose)
3. [Gestión Visual: Portainer](#3-gestión-visual-portainer)
4. [Kubernetes: Cuándo y Cómo](#4-kubernetes-cuándo-y-cómo)
5. [DevContainers para Desarrollo](#5-devcontainers-para-desarrollo)
6. [Alternativas Ligeras](#6-alternativas-ligeras)
7. [Matriz de Decisión](#7-matriz-de-decisión)

---

## 1. Fundamentos de Contenedores

### ¿Qué es un Contenedor?

Un contenedor es un paquete ejecutable que incluye:
- Tu aplicación
- Todas sus dependencias
- Runtime (Node.js, Python, Java)
- Configuración base

**Garantía:** "Si funciona en tu laptop, funcionará en producción."

### Docker vs Podman

```
CARACTERÍSTICA        | DOCKER              | PODMAN
----------------------|---------------------|--------------------
Daemon                | Sí (root)           | No (rootless)
Seguridad             | Requiere sudo       | Más seguro
Compatibilidad        | 100% ecosistema     | Compatible Docker
Compose               | docker-compose      | podman-compose
Enterprise            | Docker EE (pago)    | Gratis (Red Hat)
Recomendado para      | General, facilidad  | Enterprise, seguridad
```

**Recomendación:** Usa **Docker** para desarrollo y proyectos medianos. Usa **Podman** para Enterprise con requisitos de seguridad estrictos.

---

## 2. Docker y Docker Compose

### Dockerfile Optimizado (Multi-stage)

```dockerfile
# ===== BUILDER STAGE (Dependencias) =====
FROM node:20-alpine AS builder

WORKDIR /app

# Solo copiar package files primero (cache de layers)
COPY package*.json ./
RUN npm ci --only=production

# Copiar código fuente
COPY . .

# Build de producción
RUN npm run build

# ===== RUNNER STAGE (Ejecución) =====
FROM node:20-alpine

# Crear usuario no-root (SEGURIDAD)
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copiar solo lo necesario desde builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Cambiar a usuario no-root
USER nodejs

# Exponer puerto
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Comando de inicio
CMD ["node", "dist/main.js"]
```

**Beneficios:**
-  Imagen final ~50MB (vs ~1GB sin multi-stage)
-  No-root user (seguridad)
-  Health check integrado
-  Cache eficiente (dependencies layer)

### Docker Compose: Stack Completo

```yaml
# docker-compose.yml
version: '3.8'

services:
  # ===== API Backend =====
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: miapp-api
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@db:5432/miapp
      REDIS_URL: redis://redis:6379
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
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # ===== Frontend (Nuxt/Next) =====
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: miapp-frontend
    restart: unless-stopped
    ports:
      - "3001:3000"
    environment:
      NUXT_PUBLIC_API_URL: http://api:3000
    depends_on:
      - api
    networks:
      - app-network

  # ===== Base de Datos =====
  db:
    image: postgres:17-alpine
    container_name: miapp-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: miapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init-db:/docker-entrypoint-initdb.d
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ===== Redis (Cache/Queue) =====
  redis:
    image: redis:7-alpine
    container_name: miapp-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # ===== Nginx (Reverse Proxy) =====
  nginx:
    image: nginx:alpine
    container_name: miapp-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - certbot-etc:/etc/letsencrypt
    depends_on:
      - api
      - frontend
    networks:
      - app-network

  # ===== Certbot (SSL Automático) =====
  certbot:
    image: certbot/certbot
    container_name: miapp-certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - certbot-var:/var/lib/letsencrypt
    command: certonly --webroot --webroot-path=/var/www/html --email admin@midominio.com --agree-tos --no-eff-email -d midominio.com -d www.midominio.com

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  certbot-etc:
  certbot-var:

networks:
  app-network:
    driver: bridge
```

### Comandos Docker Compose Esenciales

```bash
# Desarrollo (con rebuild automático)
docker-compose up --build

# Producción (modo daemon)
docker-compose -f docker-compose.prod.yml up -d

# Ver logs en tiempo real
docker-compose logs -f api

# Reiniciar un servicio específico
docker-compose restart api

# Entrar a un contenedor (debugging)
docker-compose exec api sh

# Ver estado de servicios
docker-compose ps

# Parar y eliminar todo
docker-compose down

# Parar, eliminar y limpiar volúmenes
docker-compose down -v

# Escalar un servicio (ej: 3 instancias de api)
docker-compose up -d --scale api=3
```

---

## 3. Gestión Visual: Portainer

### ¿Qué es Portainer?

Portainer es una **UI web** para gestionar Docker/Kubernetes sin usar la terminal.

**Ideal para:**
- Clientes que necesitan ver el estado del sistema
- Equipos sin experiencia en DevOps
- Debugging rápido (logs, restart, stats)

### Instalación en 2 Minutos

```bash
# Crear volumen para datos de Portainer
docker volume create portainer_data

# Instalar Portainer (puerto 9443)
docker run -d \
  -p 8000:8000 \
  -p 9443:9443 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# Acceder en: https://localhost:9443
# Usuario: admin (crear contraseña en primer acceso)
```

### Funcionalidades Clave

```
FUNCIÓN                  | USO
-------------------------|----------------------------------
Containers               | Ver, iniciar, detener, logs
Images                   | Gestionar imágenes, pull/build
Networks                 | Crear/eliminar redes
Volumes                  | Backup, restore, limpiar
Stacks                   | Desplegar docker-compose.yml
Templates                | Apps pre-configuradas (WordPress, etc)
Users & Teams            | Control de acceso (RBAC)
Registry                 | Conectar a Docker Hub, ECR, etc
```

### Portainer vs Terminal

```
TAREA                   | TERMINAL                    | PORTAINER
------------------------|-----------------------------|-----------------
Ver logs                | docker logs -f container    | Click > Logs
Reiniciar contenedor    | docker restart container    | Click > Restart
Desplegar stack         | docker-compose up           | Upload YAML
Ver uso de recursos     | docker stats                | Dashboard visual
Limpiar imágenes viejas | docker image prune          | Click > Prune
```

**Cuándo usar Portainer:**
- Entregas a clientes no técnicos
- Múltiples servidores (Portainer Agent)
- Teams distribuidos

**Cuándo NO usar Portainer:**
- CI/CD automatizado (usa CLI)
- Kubernetes complejo (usa kubectl o Lens)

---

## 4. Kubernetes: Cuándo y Cómo

### ¿Necesitas Kubernetes?

```
PROYECTO          | CONTENEDORES | TRÁFICO      | KUBERNETES
------------------|--------------|--------------|------------
Landing Page      | 1            | <100 users   |  NO
Dashboard PyME    | 2-3          | <1k users    |  NO (usa Docker Compose)
E-commerce        | 5-10         | 1k-10k users |  MAYBE (usa Coolify primero)
SaaS Multi-tenant | 10+          | 10k+ users   |  SÍ
Microservicios    | 20+          | Variable     |  SÍ
```

**Regla de Oro:** Solo usa Kubernetes si tienes >5 servicios y necesitas auto-scaling.

### Kubernetes vs Docker Compose

```
CARACTERÍSTICA          | DOCKER COMPOSE       | KUBERNETES
------------------------|----------------------|--------------------
Curva de aprendizaje    | Baja (1 día)         | Alta (2-3 semanas)
Configuración           | 1 archivo YAML       | Múltiples manifests
Auto-scaling            | Manual               | Automático (HPA)
Self-healing            | restart: always      | Pods automáticos
Load Balancing          | Manual (Nginx)       | Built-in (Service)
Multi-nodo              |  No                |  Sí
Mejor para              | 1-2 servidores       | Clusters 3+ nodos
```

### Conceptos Clave de Kubernetes

```
COMPONENTE   | QUÉ ES
-------------|--------------------------------------------------
Pod          | Unidad mínima. 1+ contenedores ejecutándose juntos.
Deployment   | Define cuántas réplicas de un Pod quieres.
Service      | Endpoint estable para acceder a Pods (LoadBalancer).
Ingress      | Reverse proxy (como Nginx) para exponer servicios.
ConfigMap    | Variables de entorno (no-secretas).
Secret       | Variables de entorno encriptadas (DB passwords, API keys).
Volume       | Almacenamiento persistente (PersistentVolumeClaim).
Namespace    | Agrupación lógica (ej: prod, staging, dev).
```

### Ejemplo: Deployment Básico

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-api
  namespace: production
spec:
  replicas: 3  # 3 instancias del Pod
  selector:
    matchLabels:
      app: mi-api
  template:
    metadata:
      labels:
        app: mi-api
    spec:
      containers:
      - name: api
        image: miusuario/mi-api:v1.2.3
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secrets
              key: url
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mi-api-service
  namespace: production
spec:
  selector:
    app: mi-api
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

### Distribuciones de Kubernetes

```
DISTRIBUCIÓN   | MEJOR PARA                    | COMPLEJIDAD | COSTO
---------------|-------------------------------|-------------|--------
minikube       | Desarrollo local (laptop)     | Baja        | Gratis
k3s            | Edge, IoT, Raspberry Pi       | Baja        | Gratis
MicroK8s       | Workstations, CI/CD           | Baja        | Gratis
Kind           | Testing local                 | Baja        | Gratis
GKE (Google)   | Producción cloud managed      | Media       | $$
EKS (AWS)      | Producción cloud managed      | Media       | $$
AKS (Azure)    | Producción cloud managed      | Media       | $$
Rancher        | Multi-cluster enterprise      | Alta        | Gratis/$$
```

**Recomendaciones:**
- **Desarrollo:** minikube o Docker Desktop (incluye k8s)
- **Producción Startup:** k3s en VPS (muy ligero)
- **Producción Enterprise:** GKE/EKS/AKS

### Helm: Gestor de Paquetes de Kubernetes

Helm es como `npm` pero para Kubernetes.

```bash
# Instalar PostgreSQL con un comando
helm install my-postgres bitnami/postgresql \
  --set auth.postgresPassword=mi-password \
  --set persistence.size=10Gi

# Instalar Redis
helm install my-redis bitnami/redis

# Ver charts instalados
helm list

# Actualizar
helm upgrade my-postgres bitnami/postgresql --set auth.postgresPassword=nuevo-pass

# Desinstalar
helm uninstall my-postgres
```

**Beneficios:**
- Configuración pre-hecha
- Versionamiento (`helm rollback`)
- Reutilización de templates

---

## 5. DevContainers para Desarrollo

### ¿Qué es un DevContainer?

Un DevContainer es un **entorno de desarrollo completo** dentro de Docker.

**Beneficios:**
-  "Funciona en mi máquina" → "Funciona para todos"
-  Onboarding nuevo dev: 5 minutos
-  Mismo entorno dev = producción

### Configuración (VS Code)

```json
// .devcontainer/devcontainer.json
{
  "name": "Mi Proyecto Node.js",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  
  // Features pre-instaladas
  "features": {
    "ghcr.io/devcontainers/features/node:1": {
      "version": "20"
    },
    "ghcr.io/devcontainers/features/git:1": {},
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  
  // Extensiones de VS Code que se instalan automáticamente
  "customizations": {
    "vscode": {
      "extensions": [
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "bradlc.vscode-tailwindcss",
        "Vue.volar"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "esbenp.prettier-vscode"
      }
    }
  },
  
  // Comandos post-creación
  "postCreateCommand": "npm install",
  
  // Forward de puertos
  "forwardPorts": [3000, 5432, 6379],
  
  // Usuario no-root
  "remoteUser": "node"
}
```

```yaml
# .devcontainer/docker-compose.yml
version: '3.8'
services:
  app:
    build:
      context: ..
      dockerfile: .devcontainer/Dockerfile
    volumes:
      - ../:/workspace:cached
    command: sleep infinity
    environment:
      NODE_ENV: development
    depends_on:
      - db
      - redis

  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: dev_db
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_pass
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

volumes:
  postgres-data:
```

### Flujo de Trabajo con DevContainer

```bash
# 1. Abrir proyecto en VS Code
code .

# 2. VS Code detecta .devcontainer/
#    Popup: "Reopen in Container" → Click

# 3. Esperar 1-2 min (primera vez)
#    - Construye imagen
#    - Instala extensiones
#    - Ejecuta postCreateCommand

# 4. ¡Listo! Terminal dentro del container
npm run dev

# 5. Cerrar VS Code = detener container
# 6. Reabrir = instantáneo (usa cache)
```

**Ventajas:**
- Nuevo desarrollador: `git clone` + `Reopen in Container` = listo
- No contaminar laptop con Node 16, 18, 20 simultáneos
- Mismo setup en Windows, Mac, Linux

---

## 6. Alternativas Ligeras

### Dokploy: PaaS Auto-Hospedado (PRIORIDAD)

```
QUÉ ES: Heroku/Vercel moderno que instalas en tu propio VPS
PRECIO: Gratis (open source)
STACK: Docker + Traefik
UI: Superior a Coolify, más intuitiva
```

**Setup:**
```bash
# Instalar en VPS con Ubuntu 22.04+
curl -sSL https://dokploy.com/install.sh | sh

# Acceder en: http://tu-vps-ip:3000
# Crear aplicación desde Git repo
# Push automático = deploy automático
```

**Por qué Dokploy es PRIORIDAD:**
- UI más moderna y pulida que Coolify
- Mejor experiencia de usuario
- Actualizaciones más frecuentes
- Comunidad activa y creciente
- Documentación superior

**Cuándo usar:**
- Startups que quieren simplicidad de Heroku sin costos
- Proyectos medianos (1-10 servicios)
- No quieres aprender Kubernetes
- Necesitas UI intuitiva para clientes

### Coolify: Alternativa Sólida

```
QUÉ ES: PaaS auto-hospedado (alternativa a Dokploy)
PRECIO: Gratis (open source)
STACK: Docker + Traefik
```

**Setup:**
```bash
# Instalar en VPS con Ubuntu 22.04+
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash

# Acceder en: http://tu-vps-ip:8000
```

**Cuándo usar Coolify sobre Dokploy:**
- Necesitas features específicas de Coolify
- Prefieres su flujo de trabajo
- Ya lo tienes configurado y funciona bien

### Caprover: PaaS Simple

```bash
docker run -p 80:80 -p 443:443 -p 3000:3000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /captain:/captain caprover/caprover
```

**Comparativa:**

```
HERRAMIENTA | COMPLEJIDAD | UI    | ESCALABILIDAD | MEJOR PARA
------------|-------------|-------|---------------|------------------
Coolify     | Baja        | ★★★★☆ | Media         | Startups modernas
Dokploy     | Baja        | ★★★★★ | Media         | Devs sin DevOps
Caprover    | Baja        | ★★★☆☆ | Media         | Proyectos simples
Portainer   | Baja        | ★★★★☆ | Alta (con k8s)| Gestión visual
Kubernetes  | Alta        | ★★☆☆☆ | Muy Alta      | Enterprise/Scale
```

---

## 7. Matriz de Decisión

### ¿Qué herramienta usar?

```
ESCENARIO                              | SOLUCIÓN RECOMENDADA
---------------------------------------|----------------------------------
MVP rápido, 1-2 servicios              | Docker Compose
Startup, 3-5 servicios, <10k users     | Coolify o Dokploy
E-commerce mediano, necesita scaling   | Docker Compose + Portainer
Cliente necesita UI de gestión         | Portainer
Microservicios, >10 servicios          | Kubernetes (k3s o GKE)
Enterprise, multi-tenant, HA crítico   | Kubernetes + Helm + Rancher
Desarrollo local, equipo distribuido   | DevContainers
```

### Roadmap de Escalamiento

```
FASE 1 (0-1k usuarios):
  └─ Docker Compose en VPS único
     ├─ Nginx como reverse proxy
     └─ PostgreSQL + Redis en containers

FASE 2 (1k-10k usuarios):
  └─ Docker Compose + Portainer
     ├─ Load balancer (Nginx)
     ├─ BD separada (managed: AWS RDS)
     └─ Redis separado (ElastiCache)

FASE 3 (10k-100k usuarios):
  └─ Kubernetes (k3s o GKE)
     ├─ Auto-scaling (HPA)
     ├─ Multi-nodo (3+ workers)
     ├─ Ingress Controller (Nginx/Traefik)
     └─ Monitoring (Prometheus + Grafana)

FASE 4 (100k+ usuarios):
  └─ Kubernetes Enterprise
     ├─ Multi-region
     ├─ Service Mesh (Istio)
     ├─ Observabilidad (Datadog/New Relic)
     └─ CI/CD avanzado (ArgoCD)
```

---

## 8. Checklist Pre-Producción

```
DOCKER:
[ ] Multi-stage builds implementados
[ ] Usuario no-root en containers
[ ] Health checks configurados
[ ] Volúmenes para datos persistentes
[ ] Secrets en variables de entorno (no hardcoded)
[ ] Logs centralizados (stdout/stderr)
[ ] Resource limits definidos (memory, cpu)

DOCKER COMPOSE:
[ ] docker-compose.yml versionado en Git
[ ] .env.example documentado
[ ] Depends_on con health checks
[ ] Networks aisladas por stack
[ ] Backups automatizados de volúmenes

KUBERNETES (si aplica):
[ ] Namespaces por ambiente (prod, staging)
[ ] ResourceQuotas definidos
[ ] NetworkPolicies configuradas
[ ] Secrets en lugar de ConfigMaps para credenciales
[ ] Ingress con TLS
[ ] HPA (Horizontal Pod Autoscaler) configurado
[ ] Monitoring (Prometheus) activo
[ ] Logs centralizados (ELK o Loki)
```

---

## 9. Troubleshooting Común

### Problema: Container se reinicia constantemente

```bash
# Ver logs
docker logs -f container_name

# Ver últimas 100 líneas
docker logs --tail 100 container_name

# Inspeccionar estado
docker inspect container_name | grep -A 10 State
```

**Causas comunes:**
- Health check fallando
- OOMKilled (sin memoria)
- Comando de inicio incorrecto

### Problema: No puedo conectar entre containers

```bash
# Verificar que estén en la misma network
docker network inspect network_name

# Usar nombre del servicio (no localhost)
#  BIEN: postgresql://db:5432
#  MAL:  postgresql://localhost:5432
```

### Problema: Imagen muy grande (>1GB)

```dockerfile
# Usar alpine en lugar de debian
FROM node:20-alpine  # ~50MB
# vs
FROM node:20         # ~900MB

# Multi-stage builds
# Copiar solo node_modules de producción
COPY --from=builder /app/node_modules ./node_modules
```

---

## 10. Comandos de Supervivencia

```bash
# ===== DOCKER =====
docker ps                              # Containers activos
docker ps -a                           # Todos los containers
docker images                          # Imágenes disponibles
docker system prune -a                 # Limpiar TODO (cuidado!)
docker stats                           # Uso de recursos en tiempo real
docker exec -it container_name sh      # Entrar a container

# ===== DOCKER COMPOSE =====
docker-compose up -d                   # Iniciar en background
docker-compose down                    # Parar y eliminar
docker-compose logs -f service_name    # Logs en tiempo real
docker-compose restart service_name    # Reiniciar servicio
docker-compose pull                    # Actualizar imágenes

# ===== KUBERNETES =====
kubectl get pods                       # Ver pods
kubectl get services                   # Ver servicios
kubectl logs pod_name                  # Logs de un pod
kubectl describe pod pod_name          # Detalles de pod
kubectl exec -it pod_name -- /bin/sh   # Entrar a pod
kubectl apply -f deployment.yaml       # Aplicar cambios
kubectl rollout restart deployment/name # Reiniciar deployment
kubectl scale deployment/name --replicas=5 # Escalar a 5 réplicas
```

---

## 11. Recursos y Aprendizaje

### Documentación Oficial
- **Docker:** https://docs.docker.com/
- **Kubernetes:** https://kubernetes.io/docs/
- **Portainer:** https://docs.portainer.io/
- **Helm:** https://helm.sh/docs/

### Cursos Gratuitos
- **Docker Mastery:** https://www.youtube.com/watch?v=3c-iBn73dDE
- **Kubernetes for Beginners:** https://www.youtube.com/watch?v=X48VuDVv0do

### Herramientas Visuales
- **Lens:** https://k8slens.dev/ (IDE para Kubernetes)
- **Octant:** Dashboard local de k8s
- **k9s:** Terminal UI para k8s
