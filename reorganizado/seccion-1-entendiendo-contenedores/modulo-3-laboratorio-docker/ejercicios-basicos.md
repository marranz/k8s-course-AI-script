# Ejercicios Básicos de Docker

## Ejercicio 1: Hello World

### Objetivo
Ejecutar tu primer contenedor y entender el ciclo básico.

### Pasos
```bash
# 1. Ejecutar el contenedor hello-world
docker run hello-world

# 2. Ver qué sucedió
docker ps -a

# 3. Limpiar
docker rm <container_id>
```

### Preguntas
- ¿Qué mensaje apareció?
- ¿Por qué el contenedor aparece como "Exited"?

## Ejercicio 2: Contenedor Interactivo

### Objetivo
Trabajar dentro de un contenedor de forma interactiva.

### Pasos
```bash
# 1. Ejecutar Ubuntu de forma interactiva
docker run -it ubuntu bash

# 2. Dentro del contenedor, ejecuta:
ls
pwd
cat /etc/os-release
apt update
apt install -y curl
curl --version

# 3. Salir del contenedor
exit

# 4. Verificar el estado
docker ps -a
```

### Preguntas
- ¿Qué sucede si vuelves a ejecutar `docker run -it ubuntu bash`?
- ¿El paquete curl sigue instalado?
- ¿Por qué?

## Ejercicio 3: Contenedor en Segundo Plano

### Objetivo
Ejecutar un servidor web en segundo plano.

### Pasos
```bash
# 1. Ejecutar nginx en segundo plano
docker run -d -p 8080:80 --name mi-nginx nginx

# 2. Verificar que está corriendo
docker ps

# 3. Acceder desde el navegador
# Abrir http://localhost:8080

# 4. Ver los logs
docker logs mi-nginx
docker logs -f mi-nginx  # Ctrl+C para salir

# 5. Detener el contenedor
docker stop mi-nginx

# 6. Iniciarlo de nuevo
docker start mi-nginx

# 7. Limpieza
docker stop mi-nginx
docker rm mi-nginx
```

### Preguntas
- ¿Cuál es la diferencia entre `docker run` y `docker start`?
- ¿Qué hace la opción `-d`?
- ¿Qué significa `-p 8080:80`?

## Ejercicio 4: Ejecutar Comandos en Contenedores

### Objetivo
Interactuar con un contenedor en ejecución.

### Pasos
```bash
# 1. Iniciar un contenedor nginx
docker run -d --name web nginx

# 2. Ejecutar comandos dentro del contenedor
docker exec web ls /usr/share/nginx/html
docker exec web cat /usr/share/nginx/html/index.html

# 3. Entrar al contenedor de forma interactiva
docker exec -it web bash

# 4. Dentro del contenedor:
cd /usr/share/nginx/html
ls -la
echo "<h1>Hola desde Docker</h1>" > index.html
exit

# 5. Refrescar el navegador en http://localhost:8080
# ¿Ves el cambio?

# 6. Limpieza
docker stop web
docker rm web
```

### Preguntas
- ¿Cuál es la diferencia entre `docker run` y `docker exec`?
- ¿Qué sucede con los cambios al eliminar el contenedor?

## Ejercicio 5: Gestión de Imágenes

### Objetivo
Aprender a gestionar imágenes locales.

### Pasos
```bash
# 1. Descargar varias imágenes
docker pull nginx
docker pull nginx:alpine
docker pull python:3.9
docker pull python:3.9-slim

# 2. Listar todas las imágenes
docker images

# 3. Ver el tamaño de las imágenes
docker images --format "{{.Repository}}:{{.Tag}} - {{.Size}}"

# 4. Eliminar una imagen
docker rmi nginx:alpine

# 5. Intentar eliminar una imagen en uso
docker run -d --name test nginx
docker rmi nginx
# ¿Qué error obtienes?

# 6. Limpieza correcta
docker stop test
docker rm test
docker rmi nginx

# 7. Limpiar imágenes no utilizadas
docker image prune -a
```

### Preguntas
- ¿Por qué las imágenes alpine suelen ser más pequeñas?
- ¿Cuándo usarías una imagen slim vs una imagen completa?
- ¿Qué hace el comando `docker image prune`?

## Ejercicio 6: Inspeccionar Contenedores

### Objetivo
Aprender a obtener información detallada de contenedores.

### Pasos
```bash
# 1. Crear un contenedor
docker run -d --name web -p 8080:80 -e MI_VARIABLE="Hola Mundo" nginx

# 2. Inspeccionar el contenedor
docker inspect web

# 3. Obtener información específica
docker inspect --format='{{.NetworkSettings.IPAddress}}' web
docker inspect --format='{{.Config.Env}}' web
docker inspect --format='{{.State.Status}}' web

# 4. Ver estadísticas en tiempo real
docker stats web
# Ctrl+C para salir

# 5. Ver procesos dentro del contenedor
docker top web

# 6. Limpieza
docker stop web
docker rm web
```

### Preguntas
- ¿Qué información proporciona `docker inspect`?
- ¿Para qué sirve `docker stats`?

## Desafío Final del Módulo

Crea un contenedor que:
1. Use la imagen de Python
2. Se ejecute en segundo plano
3. Tenga el nombre "python-app"
4. Ejecute un simple servidor HTTP en el puerto 8000
5. Mapee ese puerto al 8000 del host

```bash
# Pista:
docker run -d --name python-app -p 8000:8000 python:3.9 \
  python -m http.server 8000
```

Accede a http://localhost:8000 y verifica que funciona.

## Soluciones

Las soluciones están al final de cada ejercicio. Intenta resolverlos primero por tu cuenta antes de ver las respuestas.
