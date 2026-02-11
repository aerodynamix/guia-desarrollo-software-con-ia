# 24 - Posicionamiento SEO

> Si el proyecto necesita ser encontrado en Google, el SEO no es opcional.
> Aplica a: landing pages, ecommerce, portafolios, SaaS (landing publica).

---

## Cuando Aplicar SEO

```
PROYECTO             | SEO NECESARIO
---------------------|---------------
Landing Page         | Si (critico)
Portafolio           | Si
Ecommerce            | Si (critico)
Blog                 | Si (critico)
SaaS (landing)       | Si
Dashboard            | No (app privada)
CRM                  | No
POS                  | No
Panel Admin          | No
```

---

## Meta Tags Esenciales

```html
<head>
  <!-- Titulo unico por pagina (50-60 caracteres) -->
  <title>Zapatillas Running Nike Air - Tienda Deportiva | MiTienda.cl</title>

  <!-- Descripcion unica (150-160 caracteres) -->
  <meta name="description" content="Compra zapatillas Nike Air para running.
    Envio gratis sobre $30.000. Garantia 30 dias. Todas las tallas disponibles.">

  <!-- Viewport (obligatorio para mobile) -->
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <!-- Idioma -->
  <html lang="es-CL">

  <!-- Canonical (evitar contenido duplicado) -->
  <link rel="canonical" href="https://mitienda.cl/zapatillas/nike-air">

  <!-- Robots -->
  <meta name="robots" content="index, follow">

  <!-- Open Graph (redes sociales) -->
  <meta property="og:title" content="Zapatillas Running Nike Air">
  <meta property="og:description" content="Compra zapatillas Nike Air...">
  <meta property="og:image" content="https://mitienda.cl/img/nike-air-og.jpg">
  <meta property="og:url" content="https://mitienda.cl/zapatillas/nike-air">
  <meta property="og:type" content="product">

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:title" content="Zapatillas Running Nike Air">
  <meta name="twitter:image" content="https://mitienda.cl/img/nike-air-og.jpg">

  <!-- Favicon -->
  <link rel="icon" href="/favicon.ico" sizes="32x32">
  <link rel="icon" href="/icon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
</head>
```

---

## Datos Estructurados (Schema.org / JSON-LD)

### Negocio Local

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Mi Negocio",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "Av. Providencia 1234",
    "addressLocality": "Santiago",
    "addressCountry": "CL"
  },
  "telephone": "+56912345678",
  "url": "https://minegocio.cl"
}
</script>
```

### Producto (Ecommerce)

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Zapatillas Nike Air",
  "image": "https://mitienda.cl/img/nike-air.jpg",
  "description": "Zapatillas de running Nike Air, tallas 38-45",
  "offers": {
    "@type": "Offer",
    "price": "59990",
    "priceCurrency": "CLP",
    "availability": "https://schema.org/InStock"
  }
}
</script>
```

### Breadcrumbs

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Inicio", "item": "https://mitienda.cl" },
    { "@type": "ListItem", "position": 2, "name": "Zapatillas", "item": "https://mitienda.cl/zapatillas" },
    { "@type": "ListItem", "position": 3, "name": "Nike Air" }
  ]
}
</script>
```

---

## Archivos Tecnicos

### sitemap.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://mitienda.cl/</loc>
    <lastmod>2026-01-15</lastmod>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>https://mitienda.cl/productos</loc>
    <lastmod>2026-01-15</lastmod>
    <priority>0.8</priority>
  </url>
</urlset>
```

Nuxt/Next.js/Astro generan sitemap automaticamente con plugins.

### robots.txt

```
User-agent: *
Allow: /
Disallow: /admin
Disallow: /api
Sitemap: https://mitienda.cl/sitemap.xml
```

---

## Performance y SEO

```
Google usa Core Web Vitals como factor de ranking:

LCP < 2.5s    -> Carga rapida
INP < 200ms   -> Interactividad rapida
CLS < 0.1     -> Sin saltos visuales

Ver docs/08-RENDIMIENTO.md para optimizacion detallada.
```

---

## URLs Amigables

```
MAL:   /producto?id=123&cat=5
BIEN:  /zapatillas/nike-air-max-90

Reglas:
- Minusculas
- Separar con guiones (no guion bajo)
- Sin parametros innecesarios
- Descriptivas y cortas
- Sin caracteres especiales ni tildes
```

---

## Checklist SEO Pre-Deploy

Ver [CHECKLIST-PREDEPLOY.md](../CHECKLIST-PREDEPLOY.md) -- seccion SEO.
