# 29 - Mantenimiento y Salud del Software

> "El software es como un coche. Si no le cambias el aceite, el motor se funde."

Este documento define los protocolos que la IA debe sugerir al humano para mantener el sistema operativo, seguro y eficiente a largo plazo.

---

## 1. Actualización de Dependencias (Mensual)

La "podredumbre del software" (bit rot) es real. Las dependencias envejecen y crean agujeros de seguridad.

### Protocolo de Auditoría
Ejecuta estos comandos para buscar vulnerabilidades:

**Node.js / NPM:**
```bash
# 1. Buscar vulnerabilidades conocidas
npm audit

# 2. Ver paquetes desactualizados
npm outdated

# 3. Actualizar parches seguros (Minor versions)
npm update
```

**Java / Maven:**
```bash
# Ver versiones nuevas disponibles de librerías
./mvnw versions:display-dependency-updates

# Ver plugins desactualizados
./mvnw versions:display-plugin-updates
```

**Regla IA:** Si el usuario pide "Mantenimiento", ofrece correr estos comandos y analizar la salida. NO actualices versiones `Major` (ej: v1.0 -> v2.0) sin revisar el CHANGELOG manualmente.

---

## 2. Base de Datos (Salud y Limpieza)

Las bases de datos acumulan "basura" (tuplas muertas, logs viejos).

### PostgreSQL (`VACUUM`)
Postgres necesita mantenimiento para recuperar espacio en disco de filas borradas/actualizadas.

```sql
-- Recuperar espacio (Puede bloquear tablas, usar con cuidado en prod)
VACUUM (VERBOSE, ANALYZE);

-- Reindexar si las búsquedas se vuelven lentas
REINDEX DATABASE nombre_bd;
```

### Logs y Tablas Temporales
Si tienes una tabla `logs` o `audits` que crece infinito:
1.  **Particionamiento:** Divide la tabla por meses/años.
2.  **Purga Automática:**
    ```sql
    -- Borrar logs de mas de 1 año
    DELETE FROM system_logs WHERE created_at < NOW() - INTERVAL '1 year';
    ```

---

## 3. Seguridad (Certificados y Secretos)

### Renovación SSL (HTTPS)
Si usas Certbot/LetsEncrypt, los certificados caducan cada 90 días.
*   **Comando:** `certbot renew`
*   **Cron:** Asegúrate de que exista un cron job automático.

### Rotación de Secretos (Anual)
Es buena práctica cambiar passwords de BD y API Keys una vez al año.
1.  Generar nueva API Key.
2.  Actualizar `.env` en Coolify/Servidor.
3.  Reiniciar servicio.
4.  Revocar Key vieja.

---

## 4. Backups (Verificación - Trimestral)

Tener backups automáticos no sirve si están corruptos.
**Simulacro de Incendio:**
1.  Descarga el último backup de S3.
2.  Intenta restaurarlo en tu entorno **Local**.
3.  Si falla, el backup no sirve. Arréglalo AHORA.

---

## 5. Limpieza de Disco (Docker)

Docker devora disco duro con imágenes viejas y caché.

**Comando de Limpieza (Safe):**
```bash
# Borra contenedores parados, redes sin usar e imágenes sin etiqueta
docker system prune -f
```

**Comando Nuclear (Solo si falta espacio crítico):**
```bash
# Borra TODO lo no usado (incluyendo caché de build)
docker system prune -a -f
```
