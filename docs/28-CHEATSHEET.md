# 28 - Cheatsheet de Operaciones

> "No memorices lo que puedes leer en 2 segundos."

---

## 1. Linux / Bash (Supervivencia)

### Logs y Procesos
```bash
# Ver últimas 100 líneas de un log en tiempo real
tail -f -n 100 server.log

# Buscar error específico en logs (case insensitive)
grep -i "error" server.log

# Ver uso de RAM y CPU (mejor que top)
htop

# Buscar proceso por puerto (ej: quien usa el 8080?)
lsof -i :8080
# O también:
netstat -tulpn | grep 8080

# Matar proceso pegado (a la fuerza)
kill -9 <PID>
```

### Espacio en Disco
```bash
# Ver espacio disponible
df -h

# Ver que carpetas pesan mas (ordenado)
du -sh * | sort -rh | head -10
```

### Red
```bash
# Ver mi IP pública
curl ifconfig.me

# Probar si un servicio remoto responde
telnet google.com 80
# O con curl:
curl -v https://mi-api.com/health
```

---

## 2. Java / Spring Boot

### Maven Wrapper (`./mvnw`)
Siempre usa el wrapper del proyecto, no tu Maven local. Garantiza la versión correcta.

```bash
# Limpiar y construir (sin tests para velocidad en local)
./mvnw clean package -DskipTests

# Ejecutar tests específicos
./mvnw test -Dtest=UsuarioServiceTest

# Instalar dependencias nuevas (update snapshots)
./mvnw install -U

# Ver árbol de dependencias (para conflictos de versiones)
./mvnw dependency:tree
```

### Ejecutar JAR
```bash
# Ejecutar con perfil de prod
java -jar -Dspring.profiles.active=prod app.jar

# Asignar memoria (Min 512MB, Max 1GB)
java -Xms512m -Xmx1024m -jar app.jar
```

---

## 3. Docker (Debugging)

### Contenedores
```bash
# Entrar a un contenedor corriendo (bash o sh)
docker exec -it <container_id> /bin/bash

# Ver logs de un contenedor
docker logs -f --tail 100 <container_id>

# Reiniciar contenedor
docker restart <container_id>
```

### Gestión de Contenedores y Recursos (Docker / Podman)

```bash
# Detener todos los contenedores en ejecución
docker stop $(docker ps -q)

# Eliminar todos los contenedores (debe estar detenidos)
docker rm $(docker ps -a -q)

# Eliminar todas las imágenes
docker rmi $(docker images -q)

# Limpieza general (contenedores, redes e imágenes huérfanas)
docker system prune -f

# Limpieza PROFUNDA (Incluye volúmenes de datos - CUIDADO)
# Usa esto solo si quieres borrar bases de datos y persistencia local
docker system prune -a --volumes -f

# Detener y borrar servicios de un Docker Compose
docker compose down

# Detener, borrar servicios y REVENTAR volúmenes de datos
docker compose down -v
```

### Limpieza (Cuidado!)

---

## 4. PM2 (Node.js / Python en Background)

Si no usas Docker, PM2 es el estándar para mantener procesos vivos.

```bash
# Iniciar app
pm2 start dist/main.js --name "mi-api"

# Ver estado
pm2 status
pm2 monit

# Ver logs
pm2 logs mi-api

# Guardar lista de procesos (para que revivan al reiniciar server)
pm2 save
pm2 startup
```

---

## 5. Git (Emergencia)

```bash
# Deshacer cambios en un archivo (volver al último commit)
git checkout -- nombre_archivo.ts

# Borrar cambios locales no commiteados (Hard Reset)
git reset --hard HEAD

# Forzar pull (sobrescribir local con remoto)
git fetch --all
git reset --hard origin/main
```
