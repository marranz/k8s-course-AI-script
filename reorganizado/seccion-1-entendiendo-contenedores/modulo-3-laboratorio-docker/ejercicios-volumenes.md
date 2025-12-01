# Ejercicios de Volúmenes y Persistencia

## Ejercicio 1: Crear y Usar un Volumen

### Objetivo
Entender cómo los volúmenes persisten datos más allá del ciclo de vida del contenedor.

### Pasos

```bash
# 1. Crear un volumen
docker volume create datos-ejercicio1

# 2. Listar volúmenes
docker volume ls

# 3. Inspeccionar el volumen
docker volume inspect datos-ejercicio1

# 4. Crear contenedor que usa el volumen
docker run -d --name contenedor1 \
  -v datos-ejercicio1:/data \
  alpine sleep 3600

# 5. Escribir datos en el volumen
docker exec contenedor1 sh -c "echo 'Hola desde contenedor1' > /data/mensaje.txt"

# 6. Leer los datos
docker exec contenedor1 cat /data/mensaje.txt

# 7. Eliminar el contenedor
docker rm -f contenedor1

# 8. Crear NUEVO contenedor con el MISMO volumen
docker run -d --name contenedor2 \
  -v datos-ejercicio1:/data \
  alpine sleep 3600

# 9. Verificar que los datos persisten
docker exec contenedor2 cat /data/mensaje.txt
```

### Preguntas
- ¿Qué sucedió con los datos cuando eliminaste contenedor1?
- ¿Por qué contenedor2 puede leer los datos de contenedor1?

### Limpieza
```bash
docker rm -f contenedor2
docker volume rm datos-ejercicio1
```

## Ejercicio 2: Múltiples Contenedores Compartiendo Volumen

### Objetivo
Varios contenedores pueden acceder al mismo volumen simultáneamente.

### Pasos

```bash
# 1. Crear volumen compartido
docker volume create datos-compartidos

# 2. Contenedor escritor
docker run -d --name escritor \
  -v datos-compartidos:/compartido \
  alpine sh -c 'while true; do date >> /compartido/log.txt; sleep 2; done'

# 3. Contenedor lector
docker run -d --name lector \
  -v datos-compartidos:/compartido \
  alpine sh -c 'tail -f /compartido/log.txt'

# 4. Ver logs del lector
docker logs -f lector

# 5. Detener escritor
docker stop escritor

# 6. Verificar que lector todavía tiene acceso
docker exec lector cat /compartido/log.txt
```

### Limpieza
```bash
docker rm -f escritor lector
docker volume rm datos-compartidos
```

## Ejercicio 3: Bind Mounts

### Objetivo
Usar bind mounts para desarrollo con sincronización en tiempo real.

### Pasos

```bash
# 1. Crear directorio en tu máquina
mkdir -p ~/docker-ejercicios/web
cd ~/docker-ejercicios/web

# 2. Crear un archivo HTML
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Mi Web</title></head>
<body>
  <h1>Hola desde bind mount!</h1>
  <p>Versión 1.0</p>
</body>
</html>
EOF

# 3. Montar directorio en nginx
docker run -d --name web-dev \
  -p 8080:80 \
  -v ~/docker-ejercicios/web:/usr/share/nginx/html \
  nginx:alpine

# 4. Abrir en navegador
# http://localhost:8080

# 5. Modificar index.html (sin detener contenedor)
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Mi Web</title></head>
<body>
  <h1>Hola desde bind mount!</h1>
  <p>Versión 2.0 - ACTUALIZADO</p>
</body>
</html>
EOF

# 6. Refrescar navegador - ¡Los cambios están ahí!
```

### Preguntas
- ¿Tuviste que reiniciar el contenedor para ver los cambios?
- ¿Qué ventaja tiene esto para desarrollo?

### Limpieza
```bash
docker rm -f web-dev
rm -rf ~/docker-ejercicios/web
```

## Ejercicio 4: Volúmenes vs Bind Mounts - Comparación

### Objetivo
Entender cuándo usar cada uno.

### Parte A: Volumen (Producción)

```bash
# Base de datos con volumen
docker run -d --name postgres-prod \
  -e POSTGRES_PASSWORD=secreto \
  -v datos-postgres:/var/lib/postgresql/data \
  postgres:15-alpine

# Insertar datos
docker exec -it postgres-prod psql -U postgres -c \
  "CREATE TABLE usuarios (id SERIAL, nombre VARCHAR(50));"

docker exec -it postgres-prod psql -U postgres -c \
  "INSERT INTO usuarios (nombre) VALUES ('Juan'), ('María');"

# Verificar datos
docker exec -it postgres-prod psql -U postgres -c \
  "SELECT * FROM usuarios;"

# Detener y eliminar contenedor
docker rm -f postgres-prod

# Crear NUEVO contenedor con mismo volumen
docker run -d --name postgres-prod-v2 \
  -e POSTGRES_PASSWORD=secreto \
  -v datos-postgres:/var/lib/postgresql/data \
  postgres:15-alpine

# Verificar que datos persisten
sleep 5
docker exec -it postgres-prod-v2 psql -U postgres -c \
  "SELECT * FROM usuarios;"
```

### Parte B: Bind Mount (Desarrollo)

```bash
# Crear directorio para config
mkdir -p ~/docker-ejercicios/postgres-config

# Crear archivo de configuración personalizada
cat > ~/docker-ejercicios/postgres-config/custom.conf << 'EOF'
# Configuración personalizada
max_connections = 200
shared_buffers = 256MB
EOF

# Postgres con configuración custom
docker run -d --name postgres-dev \
  -e POSTGRES_PASSWORD=secreto \
  -v ~/docker-ejercicios/postgres-config/custom.conf:/etc/postgresql/postgresql.conf:ro \
  postgres:15-alpine -c config_file=/etc/postgresql/postgresql.conf

# Verificar configuración
docker exec postgres-dev cat /etc/postgresql/postgresql.conf
```

### Limpieza
```bash
docker rm -f postgres-prod-v2 postgres-dev
docker volume rm datos-postgres
rm -rf ~/docker-ejercicios/postgres-config
```

## Ejercicio 5: Backup y Restore de Volúmenes

### Objetivo
Aprender a hacer backup y restaurar datos de volúmenes.

### Pasos

```bash
# 1. Crear volumen con datos
docker volume create datos-importante

# 2. Llenar con datos
docker run --rm \
  -v datos-importante:/data \
  alpine sh -c 'echo "Información crítica" > /data/importante.txt && \
                echo "Más datos" > /data/mas-datos.txt'

# 3. BACKUP - Crear tar del volumen
docker run --rm \
  -v datos-importante:/source:ro \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup-datos.tar.gz -C /source .

# 4. Verificar backup
ls -lh backup-datos.tar.gz

# 5. Eliminar volumen original
docker volume rm datos-importante

# 6. RESTORE - Crear nuevo volumen
docker volume create datos-restaurados

# 7. Restaurar desde backup
docker run --rm \
  -v datos-restaurados:/target \
  -v $(pwd):/backup \
  alpine sh -c 'cd /target && tar xzf /backup/backup-datos.tar.gz'

# 8. Verificar datos restaurados
docker run --rm \
  -v datos-restaurados:/data \
  alpine ls -la /data

docker run --rm \
  -v datos-restaurados:/data \
  alpine cat /data/importante.txt
```

### Limpieza
```bash
docker volume rm datos-restaurados
rm backup-datos.tar.gz
```

## Ejercicio 6: Volúmenes Read-Only

### Objetivo
Proteger datos montando volúmenes como solo lectura.

### Pasos

```bash
# 1. Crear volumen con configuración
docker volume create config-app

# 2. Escribir configuración
docker run --rm \
  -v config-app:/config \
  alpine sh -c 'echo "API_KEY=secret123" > /config/app.conf'

# 3. Contenedor con volumen READ-ONLY
docker run -d --name app-segura \
  -v config-app:/config:ro \
  alpine sh -c 'while true; do cat /config/app.conf; sleep 10; done'

# 4. Intentar modificar (debería fallar)
docker exec app-segura sh -c 'echo "HACK" > /config/app.conf'
# Error: Read-only file system

# 5. Ver logs para confirmar que sigue leyendo
docker logs app-segura
```

### Limpieza
```bash
docker rm -f app-segura
docker volume rm config-app
```

## Ejercicio 7: Limpieza de Volúmenes Huérfanos

### Objetivo
Gestionar volúmenes no utilizados.

### Pasos

```bash
# 1. Crear varios volúmenes
docker volume create volumen-usado
docker volume create volumen-huerfano1
docker volume create volumen-huerfano2

# 2. Solo usar uno
docker run -d --name usa-volumen \
  -v volumen-usado:/data \
  alpine sleep 3600

# 3. Listar todos los volúmenes
docker volume ls

# 4. Ver volúmenes huérfanos (sin usar)
docker volume ls -f dangling=true

# 5. Eliminar volúmenes no usados
docker volume prune

# Confirmar: y

# 6. Verificar que solo queda el usado
docker volume ls
```

### Limpieza
```bash
docker rm -f usa-volumen
docker volume rm volumen-usado
```

## Ejercicio 8: Proyecto Práctico - Blog con Persistencia

### Objetivo
Aplicación completa con base de datos persistente.

### Pasos

```bash
# 1. Crear red personalizada
docker network create blog-network

# 2. Crear volumen para MySQL
docker volume create blog-mysql-data

# 3. Contenedor MySQL
docker run -d \
  --name blog-db \
  --network blog-network \
  -v blog-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppass \
  mysql:8.0

# 4. Esperar que MySQL esté listo
sleep 20

# 5. Volumen para WordPress
docker volume create blog-wp-data

# 6. Contenedor WordPress
docker run -d \
  --name blog-wp \
  --network blog-network \
  -p 8080:80 \
  -v blog-wp-data:/var/www/html \
  -e WORDPRESS_DB_HOST=blog-db \
  -e WORDPRESS_DB_NAME=wordpress \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppass \
  wordpress:latest

# 7. Abrir navegador
# http://localhost:8080
# Completa instalación de WordPress

# 8. Crear un post de prueba

# 9. SIMULAR DESASTRE - Eliminar ambos contenedores
docker rm -f blog-wp blog-db

# 10. RECUPERACIÓN - Recrear contenedores con MISMOS volúmenes
docker run -d \
  --name blog-db \
  --network blog-network \
  -v blog-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=rootpass \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wpuser \
  -e MYSQL_PASSWORD=wppass \
  mysql:8.0

sleep 20

docker run -d \
  --name blog-wp \
  --network blog-network \
  -p 8080:80 \
  -v blog-wp-data:/var/www/html \
  -e WORDPRESS_DB_HOST=blog-db \
  -e WORDPRESS_DB_NAME=wordpress \
  -e WORDPRESS_DB_USER=wpuser \
  -e WORDPRESS_DB_PASSWORD=wppass \
  wordpress:latest

# 11. Verificar en navegador - ¡Tu post sigue ahí!
```

### Limpieza
```bash
docker rm -f blog-wp blog-db
docker network rm blog-network
docker volume rm blog-mysql-data blog-wp-data
```

## Desafío Final

### Objetivo
Sistema completo con backup automático.

### Requisitos
1. Contenedor de aplicación con volumen de datos
2. Contenedor que hace backup cada 1 minuto
3. Script que restaura desde último backup

### Pistas
```bash
# Backup automático con cron en contenedor
docker run -d --name backup-automatico \
  -v datos-app:/source:ro \
  -v $(pwd)/backups:/backup \
  alpine sh -c 'while true; do \
    tar czf /backup/backup-$(date +%Y%m%d-%H%M%S).tar.gz -C /source . && \
    sleep 60; \
  done'
```

## Resumen de Comandos

```bash
# Volúmenes
docker volume create <nombre>
docker volume ls
docker volume inspect <nombre>
docker volume rm <nombre>
docker volume prune

# Usar volúmenes
docker run -v <volumen>:<path-contenedor> <imagen>
docker run -v <volumen>:<path>:ro <imagen>  # Read-only

# Bind mounts
docker run -v <path-host>:<path-contenedor> <imagen>
docker run -v $(pwd):/app <imagen>

# Backup
docker run --rm -v <vol>:/source -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /source .

# Restore
docker run --rm -v <vol>:/target -v $(pwd):/backup alpine \
  sh -c 'cd /target && tar xzf /backup/backup.tar.gz'
```

## Mejores Prácticas

1. ✅ Usa **volúmenes** para datos de producción
2. ✅ Usa **bind mounts** para desarrollo
3. ✅ Monta configuraciones como **read-only** (`:ro`)
4. ✅ Haz **backups regulares** de volúmenes críticos
5. ✅ Limpia volúmenes huérfanos periódicamente
6. ✅ Nombra volúmenes descriptivamente
7. ✅ Documenta qué volúmenes necesita tu aplicación
