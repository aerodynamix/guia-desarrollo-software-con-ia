# 11 - Despliegue y DevOps

> "Funciona en mi máquina" no es excusa válida.
> El despliegue debe ser predecible, automatizado y aburrido.

---

## 1. Estrategia de Ambientes

| Ambiente | Rama | URL | Propósito | Datos |
|----------|------|-----|-----------|-------|
| **Local** | `feature/*` | localhost:4200 | Desarrollo activo | Seeds/Fake |
| **Dev** | `develop` | dev.midominio.com | Integración continua | Fake/Snapshot |
| **Staging** | `release/*` | qa.midominio.com | Réplica de Prod | Copia sanitizada |
| **Producción** | `main` | midominio.com | Mundo real | Reales |

**REGLA:** Nunca deploys directos a producción desde local. Siempre vía CI/CD.

---

## 2. OPCIONES DE DEPLOY (PRIORIDAD)

### OPCIÓN 1 (RECOMENDADA): Dokploy en VPS

**Dokploy** es la solución PaaS self-hosted IDEAL para freelancers y PyMEs. Funciona sobre Docker y Traefik.

**VENTAJAS:**
- Gratuito y open-source
- UI moderna y limpia
- Deploy automático desde Git (Webhook)
- SSL automático (Let's Encrypt)
- Base de datos integrada (PostgreSQL, Redis)

**STACK TÍPICO DOKPLOY:**
- **App:** Dockerfile (Angular + Nginx)
- **API:** Dockerfile (NestJS)
- **BD:** PostgreSQL 17 (Managed by Dokploy)
- **Redis:** Redis 7 (Managed by Dokploy)

**COSTO:** VPS Hetzner CPX11 (~$5 USD/mes).

---

### OPCIÓN 2: Coolify (Alternativa)
Similar a Dokploy, más maduro pero con UI más compleja. Excelente opción si necesitas gestionar múltiples servidores desde un solo panel.

### OPCIÓN 3: Vercel (Solo Frontend)
Ideal para el Frontend Angular si el backend está en otro lado (ej: Railway, AWS).
- **Ventaja:** CDN global, Edge Network, CI/CD zero-config.
- **Costo:** Gratis (Hobby) / $20 mes (Pro).

---

## 3. DOCKERFILE OPTIMIZADO (Angular)

Usar **Multi-stage build** para mantener la imagen ligera.

```dockerfile
# STAGE 1: Build
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build -- --configuration production

# STAGE 2: Runtime (Nginx)
FROM nginx:alpine

# Copiar build de Angular a Nginx
COPY --from=builder /app/dist/mi-app/browser /usr/share/nginx/html

# Copiar configuracion personalizada Nginx (para SPA routing)
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf (Requerido para SPA):**
```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 4. DOCKERFILE OPTIMIZADO (NestJS)

```dockerfile
# STAGE 1: Build
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# STAGE 2: Runtime
FROM node:20-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./

# Usuario no-root (seguridad)
USER node

EXPOSE 3000
CMD ["node", "dist/main"]
```

---

## 5. CI/CD CON GITHUB ACTIONS

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Trigger Dokploy Webhook
        run: |
          curl -X POST "${{ secrets.DOKPLOY_WEBHOOK_URL }}"
```

---

## 6. VARIABLES DE ENTORNO (SEGURIDAD)

**NUNCA** comitees `.env` a Git.

**ESTRUCTURA:**
```
.env.example       ← Comitear (sin valores)
.env               ← NO comitear (valores reales)
```

**EJEMPLO .env.example:**
```bash
# Base de Datos
DATABASE_URL=postgresql://user:password@db:5432/dbname
REDIS_URL=redis://redis:6379

# JWT
JWT_SECRET=your-secret-here
JWT_EXPIRES_IN=1h

# Frontend URL (CORS)
FRONTEND_URL=https://mi-app.com
```

---

## 7. BACKUPS

**Regla 3-2-1:** 3 copias, 2 medios, 1 offsite.

**Script de Backup (PostgreSQL):**
```bash
#!/bin/bash
docker exec postgres_container pg_dump -U postgres mydb > backup_$(date +%F).sql
# Subir a S3 / R2...
```
Dokploy y Coolify tienen sistemas de backup automatizados a S3/R2. **ACTÍVALOS.**

---

## 8. CHECKLIST PRE-DEPLOY

- [ ] Variables de entorno configuradas en el servidor.
- [ ] SSL/HTTPS activo y forzado.
- [ ] Base de datos migrada (`npm run typeorm migration:run`).
- [ ] Usuarios semilla creados (si es primer deploy).
- [ ] Logs monitoreados (sin errores 500 al inicio).
