# Checklist Unificado Pre-Deploy

> Revisar ANTES de cada despliegue a produccion.
> No todos los items aplican a todos los proyectos. Marcar N/A si no corresponde.

---

## Infraestructura

```
[ ] Dockerfile multi-stage optimizado
[ ] docker-compose.yml para dev y prod
[ ] Variables de entorno configuradas en servidor (no hardcodeadas)
[ ] .env en .gitignore, .env.example actualizado
[ ] SSL/HTTPS activo (Let's Encrypt / proveedor)
[ ] Nginx como reverse proxy configurado
[ ] Firewall activo (solo puertos 80, 443, SSH)
[ ] Dominio configurado (DNS A record)
[ ] CI/CD configurado (opcional pero recomendado)
[ ] Monitoreo basico (Uptime Robot, /health endpoint)
[ ] Backups automaticos de BD programados
```

## Seguridad

```
[ ] CORS configurado (no wildcard en produccion)
[ ] Headers de seguridad activos (Helmet o manual)
[ ] Passwords hasheados con bcrypt/argon2
[ ] SQL parametrizado o via ORM
[ ] Rate limiting activo en login, registro y API publica
[ ] Validacion de datos en frontend Y backend
[ ] Tokens de sesion en cookies HttpOnly + Secure + SameSite
[ ] Dependencias auditadas (npm audit / composer audit)
[ ] CSP (Content Security Policy) configurado
[ ] Secrets rotados (no usar los de desarrollo)
```

## Rendimiento

```
[ ] Imagenes optimizadas (WebP/AVIF, lazy loading, dimensiones explicitas)
[ ] Bundle size analizado (< 200KB JS inicial ideal)
[ ] Fuentes optimizadas (preload, font-display: swap)
[ ] Lighthouse performance score > 90
[ ] Indices de BD creados para columnas en WHERE/JOIN/ORDER BY
[ ] Paginacion implementada en todos los listados
[ ] Caching configurado (HTTP Cache-Control + app cache si aplica)
[ ] Compresion activa (gzip o brotli en servidor)
[ ] Sin N+1 queries (verificar con logs o EXPLAIN)
[ ] Code splitting activo (lazy loading de rutas/modulos)
```

## SEO (solo proyectos publicos)

```
[ ] Titulo unico por pagina (50-60 caracteres)
[ ] Meta description unica (150-160 caracteres)
[ ] Etiqueta canonical en todas las paginas
[ ] Open Graph tags configurados
[ ] Schema.org/JSON-LD segun tipo de contenido
[ ] sitemap.xml generado
[ ] robots.txt configurado
[ ] URLs amigables (kebab-case, sin parametros innecesarios)
[ ] Imagenes con alt descriptivo
[ ] HTML semantico (h1 unico por pagina, jerarquia h1-h6)
[ ] Responsive verificado en 375px (mobile)
[ ] Google Search Console configurado
```

## Codigo

```
[ ] Sin console.log de debug
[ ] Sin imports no utilizados
[ ] Sin variables sin referencia
[ ] Sin funciones vacias o codigo comentado
[ ] Sin TODO/FIXME pendientes criticos
[ ] Linter pasando sin errores
[ ] Tests criticos pasando (unit + integracion)
[ ] README del proyecto actualizado con instrucciones de instalacion
[ ] Swagger/OpenAPI actualizado (si tiene API)
```

---

## Que Checklist Aplica Segun Proyecto

```
PROYECTO          | INFRA | SEGURIDAD | RENDIMIENTO | SEO | CODIGO
------------------|-------|-----------|-------------|-----|-------
Landing Page      | Parcial| No       | Si          | Si  | Si
Ecommerce         | Si    | Si        | Si          | Si  | Si
Dashboard         | Si    | Si        | Si          | No  | Si
CRM               | Si    | Si        | Si          | No  | Si
POS               | Si    | Si        | Si          | No  | Si
SaaS              | Si    | Si        | Si          | Parcial | Si
App Movil (API)   | Si    | Si        | Si          | No  | Si
```
