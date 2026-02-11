# 27 - Gestión de Archivos y Storage

> "Los archivos de usuario no son código. No los guardes en el repositorio."

---

## 1. Estrategia de Almacenamiento

El error #1 es guardar uploads en la carpeta `src/assets`. **Nunca hagas esto.**
El repositorio es efímero. Los datos de usuario son persistentes.

### Opción A: Almacenamiento Local (Intranet / MVP)
Usa **Docker Volumes**. Mapea una carpeta del host al contenedor.

*   **Ventaja:** Gratis, rápido, sin configuración de red.
*   **Desventaja:** Difícil de escalar horizontalmente (si tienes 2 servidores, los archivos están solo en uno).

**Docker Compose:**
```yaml
services:
  api:
    volumes:
      - ./uploads:/app/uploads
```

### Opción B: Cloud Storage (S3 / R2 / Azure Blob)
**Estándar para Producción.**

*   **AWS S3:** El estándar. Caro si no se optimiza.
*   **Cloudflare R2:** Compatible con S3 API. **Sin costos de egreso (bandwidth).** Recomendado.
*   **Azure Blob Storage:** Si estás en el ecosistema Microsoft.
*   **MinIO:** Self-hosted S3 compatible (si quieres "Cloud" pero en tu propio servidor).

---

## 2. Ciclo de Vida del Archivo

### A. Subida (Upload)
No subas el archivo directo al backend si es muy grande (>10MB). Usa **Presigned URLs**.

1.  **Frontend:** Pide permiso al Backend ("Quiero subir `foto.jpg`").
2.  **Backend:** Genera una URL firmada de S3 (válida por 5 min).
3.  **Frontend:** Sube el archivo directo a la nube (PUT a la URL firmada).
4.  **Backend:** Guarda la referencia (URL) en la BD.

*Beneficio:* No bloqueas el hilo de Node.js procesando streams de archivos gigantes.

### B. Edición (Reemplazo Atómico)
Nunca "modifiques" un archivo. **Reemplázalo.**

**Flujo Correcto:**
1.  Usuario sube `imagen_nueva.jpg`.
2.  Backend sube `imagen_nueva.jpg` al storage.
3.  Backend actualiza el registro en BD apuntando a la nueva URL.
4.  Backend **elimina** `imagen_vieja.jpg` del storage (o la marca para borrado asíncrono).

> **Por qué:** Evita problemas de caché de navegador. Si sobreescribes `avatar.jpg` con otro `avatar.jpg`, el usuario seguirá viendo el viejo por días.

### C. Eliminación
Cuando se borra un registro en BD (ej: Producto), se debe borrar su imagen asociada.
Usa **Eventos de Dominio** (`ProductDeleted`) para disparar un Job que limpie el storage.

---

## 3. Seguridad y Validación

### Validación de Tipos (MIME Types)
No confíes en la extensión `.jpg`. Un atacante puede renombrar `virus.exe` a `foto.jpg`.
Usa librerías que lean los **Magic Numbers** (cabeceras binarias) del archivo.

*   **Node:** `file-type` o `mmmagic`.
*   **PHP:** `finfo_file`.
*   **Python:** `python-magic`.
*   **Java:** `Apache Tika` (Estándar industrial).

### Renombrado (UUID)
Nunca guardes el archivo con el nombre original del usuario.
*   Usuario sube: `Vacaciones Verano 2026 (1) final.jpg`
*   Tú guardas: `users/123/avatars/01J2... (UUID v7).jpg`

**Beneficios:**
1.  Evitas caracteres raros (tildes, espacios, emojis) que rompen URLs.
2.  Evitas colisiones (dos usuarios subiendo `foto.jpg`).
3.  Previene enumeración (nadie puede adivinar `foto_1.jpg`, `foto_2.jpg`).

---

## 4. Servir Archivos (Delivery)

### Nunca expongas el bucket público directamente
Pon siempre una **CDN** (Cloudflare / CloudFront) delante.
*   **Cache:** Ahorra costos de requests a S3.
*   **Seguridad:** Oculta la estructura de tu bucket.
*   **Optimización:** La CDN puede convertir imágenes a WebP/AVIF al vuelo.

---

## 5. Resumen de Herramientas

| **Herramienta** | **Uso** |
|---|---|
| **Multer (Node)** | Middleware para manejar `multipart/form-data`. |
| **Sharp (Node)** | Procesamiento de imágenes (resize, WebP). |
| **Flysystem (PHP)** | Abstracción de filesystem (S3/Local) en Laravel. |
| **Spring Cloud AWS (Java)** | Integración nativa con S3 para Spring Boot. |
| **Apache Tika (Java)** | Detección de Mime-types y extracción de texto. |
| **Thumbnailator (Java)** | Redimensionamiento de imágenes simple y potente. |
| **AWS SDK** | Librería oficial para S3/R2 en todos los lenguajes. |
