# 15 - Backups y Recuperación

> Si no tienes backup, no tienes datos. Un solo fallo y pierdes todo.

---

## Estrategia de Backups (Regla 3-2-1)

```
3 copias de los datos
2 medios de almacenamiento diferentes
1 copia fuera del servidor (offsite)
```

---

## Backup de Base de Datos

### PostgreSQL

```bash
# Backup completo (dump SQL)
pg_dump -U usuario -h localhost -d mi_base -F c -f backup_$(date +%Y%m%d_%H%M%S).dump

# Restaurar
pg_restore -U usuario -h localhost -d mi_base backup_20260115_120000.dump

# Backup solo datos (sin estructura)
pg_dump -U usuario -d mi_base --data-only -f datos.sql

# Backup solo estructura (sin datos)
pg_dump -U usuario -d mi_base --schema-only -f estructura.sql
```

### MySQL

```bash
# Backup
mysqldump -u usuario -p mi_base > backup_$(date +%Y%m%d_%H%M%S).sql

# Restaurar
mysql -u usuario -p mi_base < backup_20260115_120000.sql
```

### SQLite

```bash
# Simplemente copiar el archivo
cp mi_base.db backup_mi_base_$(date +%Y%m%d).db
```

### MongoDB

```bash
# Backup
mongodump --db mi_base --out backup_$(date +%Y%m%d)

# Restaurar
mongorestore --db mi_base backup_20260115/mi_base
```

---

## Automatización con Cron (Linux)

```bash
# Editar crontab
crontab -e

# Backup diario a las 3:00 AM
0 3 * * * /home/deployer/scripts/backup.sh >> /var/log/backup.log 2>&1

# Backup semanal (domingos 2:00 AM)
0 2 * * 0 /home/deployer/scripts/backup-full.sh >> /var/log/backup.log 2>&1
```

### Script de Backup (`backup.sh`)

```bash
#!/bin/bash
set -e

# Variables
DB_NAME="mi_base"
DB_USER="usuario"
BACKUP_DIR="/home/deployer/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Crear directorio si no existe
mkdir -p $BACKUP_DIR

# Hacer backup
pg_dump -U $DB_USER -F c $DB_NAME > "$BACKUP_DIR/backup_$DATE.dump"

# Comprimir
gzip "$BACKUP_DIR/backup_$DATE.dump"

# Eliminar backups mayores a 30 dias
find $BACKUP_DIR -name "*.dump.gz" -mtime +$RETENTION_DAYS -delete

# Copiar a storage externo (S3, R2, etc.)
# aws s3 cp "$BACKUP_DIR/backup_$DATE.dump.gz" s3://mi-bucket/backups/

echo "Backup completado: backup_$DATE.dump.gz"
```

---

## Storage para Backups Offsite

| Servicio | Precio | Nota |
|----------|--------|------|
| Cloudflare R2 | 10GB gratis, $0.015/GB | Sin egress fees |
| AWS S3 Glacier | $0.004/GB | Para archivos de largo plazo |
| Backblaze B2 | 10GB gratis, $0.005/GB | Barato y simple |
| Hetzner Storage Box | 1TB por ~$4/mes | Buen precio por volumen |

---

## Frecuencia Recomendada

```
TIPO DE PROYECTO    | FRECUENCIA BD      | RETENCION
--------------------|--------------------|-----------
Landing Page        | No aplica          | -
Ecommerce           | Cada 6 horas       | 30 dias
Dashboard/CRM       | Diario             | 30 dias
POS                 | Cada 6 horas       | 60 dias
SaaS                | Cada 4-6 horas     | 90 dias
Sistema critico     | Cada hora          | 90+ dias
```

---

## Backup del Codigo Fuente

```
El codigo ya esta respaldado en Git (GitHub/GitLab).

Adicional:
- Proteger rama main (no push directo).
- Activar branch protection en GitHub.
- Tener al menos 2 personas con acceso admin al repo.
```

---

## Backup de Archivos de Usuario (Uploads)

```
Si el sistema maneja archivos subidos (imagenes, PDFs, etc.):

1. Almacenar en object storage (S3, R2, MinIO), NO en el servidor.
2. Object storage ya tiene redundancia incorporada.
3. Habilitar versionado en el bucket para recuperar archivos borrados.
4. Hacer backup del bucket a otra region (opcional pero recomendado).
```

---

## Plan de Recuperación ante Desastres (DRP)

### Métricas Objetivo
*   **RTO (Tiempo Objetivo de Recuperación):** < 4 Horas. (Tiempo máximo que el sistema puede estar caído).
*   **RPO (Punto Objetivo de Recuperación):** < 1 Hora. (Máxima cantidad de datos que aceptamos perder).

### Escenario 1: Borrado Accidental / Error Humano
*   **Síntoma:** Un dev borró la tabla `users` o un script corrompió datos.
*   **Acción:** Restaurar BD desde el último backup horario/diario.
*   **Pérdida:** Datos desde el último backup hasta el error.

### Escenario 2: Fallo Total del Servidor (Hardware/Cloud)
*   **Sintoma:** El VPS no responde o el datacenter se incendio.
*   **Accion:**
    1. Aprovisionar nuevo VPS (usar script `setup.sh` + Terraform/Ansible si hay).
    2. Instalar Docker/Coolify.
    3. Restaurar BD desde backup remoto (S3/R2).
    4. Desplegar codigo desde Git (`main`).
    5. Cambiar DNS (Cloudflare) a la nueva IP.

### Escenario 3: Ransomware / Hackeo Destructivo
*   **Sintoma:** Archivos encriptados, mensaje de extorsion, acceso denegado.
*   **Accion:**
    1. **NO PAGAR.**
    2. Formatear/Eliminar servidor infectado (Fase de Erradicacion).
    3. Crear infraestructura nueva desde cero (Fase de Recuperacion).
    4. Restaurar backups **OFFSITE** (que no estaban conectados al servidor infectado).
    5. Verificar integridad de los datos restaurados antes de abrir al publico.

```
ESCENARIO                  | ACCION
---------------------------|-------------------------------------------
BD corrupta                | Restaurar ultimo backup valido
Servidor caido             | Levantar nuevo VPS + restaurar backup
Borrado accidental datos   | Restaurar backup + replay de logs WAL
Hackeo                     | Aislar servidor, restaurar backup limpio
Error en migracion         | Revertir migracion, restaurar backup pre-migracion
```

### Probar Restauracion

```
CRITICO: Un backup que no se prueba NO es un backup.

Cada mes:
1. Tomar el backup mas reciente.
2. Restaurarlo en un ambiente de prueba.
3. Verificar que los datos estan completos.
4. Documentar el resultado.
```
