# 12 - Infraestructura y Servidores

> La infraestructura es codigo. No configures servidores manualmente.

---

## Rendering: SSR vs CSR vs SSG

¿Como se renderiza tu web? La decision define tu infraestructura.

| Modo | Nombre | Ejemplo | Infra | Mejor para |
|------|--------|---------|-------|------------|
| **CSR** | Client-Side Rendering | React, Vue (SPA) | S3 / CDN | Dashboards, Apps privadas, SaaS. |
| **SSR** | Server-Side Rendering | Next.js, Nuxt | Node.js Server | Ecommerce, Redes Sociales (SEO Critico). |
| **SSG** | Static Site Gen | Astro, Hugo | S3 / CDN | Blogs, Documentacion, Landing Pages. |

**Regla de Oro:**
*   Si necesitas SEO -> **SSR** (o SSG si cambia poco).
*   Si es detras de login -> **CSR** (Mas barato, mas facil).

---

## Proxy Inverso (Gateway)

Nunca expongas tu Node.js/Python directo a internet (Puerto 3000). Usa un Proxy Inverso enfrente.

### ¿Por que?
1.  **SSL/TLS:** Maneja los certificados HTTPS.
2.  **Seguridad:** Oculta la tecnologia del backend.
3.  **Carga:** Puede balancear carga y comprimir (Gzip/Brotli).

### Herramientas
*   **Nginx:** El estandar clasico. Robusto, rapido.
*   **Caddy:** Configuración automatica de HTTPS. Muy facil.
*   **Traefik:** Cloud-native. Detecta contenedores Docker automaticamente. (Ideal para Coolify/Dokploy).

---

## DNS y CDN (Cloudflare)

No expongas la IP real de tu servidor. Usa Cloudflare en modo "Proxy" (Nube Naranja).

### Ventajas de Cloudflare Proxy
1.  **Oculta tu IP:** Los atacantes ven la IP de Cloudflare, no la de tu VPS.
2.  **DDoS Protection:** Bloquea ataques volumetricos gratis.
3.  **Caching:** Guarda estaticos (imagenes, css) en el borde (Edge), ahorrando ancho de banda.
4.  **SSL Universal:** HTTPS gratis y automatico.

### Configuracion Correcta
*   **A Record:** `midominio.com` -> IP_VPS (Nube Naranja Activada).
*   **CNAME:** `www` -> `midominio.com` (Nube Naranja Activada).
*   **SSL Mode:** "Full (Strict)". Requiere certificado en el VPS (Let's Encrypt o Certificado Origen de Cloudflare).

---

## Dominios y Certificados

### Compra de Dominios
Evita revendedores caros. Usa registradores transparentes con precios de renovacion estables.

1.  **Cloudflare Registrar:**
    *   **Pros:** Precio mayorista (sin markup), privacidad WHOIS gratis, seguridad integrada.
    *   **Uso:** La mejor opcion si ya usas Cloudflare DNS (recomendado).
2.  **Namecheap:**
    *   **Pros:** Soporte excelente, precios bajos en primer año, interfaz sencilla.
    *   **Uso:** Buena alternativa si Cloudflare no soporta la extension TLD especifica (.io, .ai, etc).

### Certificados SSL (HTTPS)
Nunca pagues por un certificado SSL basico.

1.  **Let's Encrypt:**
    *   Estandar de la industria. Gratuito. Renovacion automatica cada 90 dias.
    *   Implementacion: `Certbot` en Nginx o automatico en Caddy/Traefik.
2.  **Cloudflare Origin Certificate:**
    *   Certificado de larga duracion (15 años) valido SOLO entre Cloudflare y tu Servidor.
    *   Ideal para modo "Full (Strict)" sin configurar renovaciones constantes en el servidor.
