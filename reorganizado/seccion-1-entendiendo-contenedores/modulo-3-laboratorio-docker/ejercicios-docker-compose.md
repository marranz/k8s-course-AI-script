# Ejercicios de Docker Compose

## Ejercicio 1: Primera Aplicación con Compose

### Objetivo
Crear tu primer `docker-compose.yml` para una aplicación simple.

### Estructura del Proyecto
```
ejercicio1/
├── docker-compose.yml
└── index.html
```

### Pasos

```bash
# 1. Crear directorio
mkdir -p ~/docker-ejercicios/compose-ej1
cd ~/docker-ejercicios/compose-ej1

# 2. Crear index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>Docker Compose</title></head>
<body>
  <h1>¡Mi primera aplicación con Docker Compose!</h1>
</body>
</html>
EOF

# 3. Crear docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
EOF

# 4. Iniciar
docker compose up -d

# 5. Verificar
curl http://localhost:8080

# 6. Ver logs
docker compose logs

# 7. Detener
docker compose down
```

### Comandos Básicos
```bash
docker compose up         # Iniciar (foreground)
docker compose up -d      # Iniciar (background)
docker compose down       # Detener y eliminar
docker compose ps         # Listar servicios
docker compose logs       # Ver logs
docker compose logs -f    # Seguir logs
```

## Ejercicio 2: Aplicación con Base de Datos

### Objetivo
WordPress con MySQL usando Compose.

### docker-compose.yml

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db

volumes:
  db_data:
  wp_data:
```

### Pasos

```bash
# 1. Crear archivo
mkdir -p ~/docker-ejercicios/compose-wordpress
cd ~/docker-ejercicios/compose-wordpress

# Copiar el docker-compose.yml de arriba

# 2. Iniciar
docker compose up -d

# 3. Esperar que esté listo
docker compose logs -f

# 4. Abrir navegador
# http://localhost:8080

# 5. Ver servicios corriendo
docker compose ps

# 6. Ejecutar comandos en servicios
docker compose exec db mysql -u root -p
# Contraseña: rootpassword

# 7. Detener (mantiene datos)
docker compose stop

# 8. Reiniciar
docker compose start

# 9. Eliminar TODO (incluyendo volúmenes)
docker compose down -v
```

## Ejercicio 3: Variables de Entorno con .env

### Objetivo
Externalizar configuración en archivo `.env`.

### Estructura
```
ejercicio3/
├── docker-compose.yml
└── .env
```

### .env
```env
# Base de datos
DB_ROOT_PASSWORD=supersecret
DB_NAME=myapp
DB_USER=appuser
DB_PASSWORD=apppass

# Aplicación
APP_PORT=3000
APP_ENV=development
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_ROOT_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  app:
    image: node:18-alpine
    ports:
      - "${APP_PORT}:3000"
    environment:
      NODE_ENV: ${APP_ENV}
      DB_HOST: db
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    command: sleep infinity

volumes:
  postgres_data:
```

### Pasos
```bash
# 1. Crear archivos
mkdir -p ~/docker-ejercicios/compose-env
cd ~/docker-ejercicios/compose-env

# Crear .env y docker-compose.yml

# 2. Verificar configuración
docker compose config

# 3. Iniciar
docker compose up -d

# 4. Verificar variables
docker compose exec app env | grep APP_
docker compose exec app env | grep DB_
```

## Ejercicio 4: Build de Imagen Custom

### Objetivo
Construir imagen dentro de Compose.

### Estructura
```
ejercicio4/
├── docker-compose.yml
├── Dockerfile
├── app.py
└── requirements.txt
```

### app.py
```python
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    env = os.getenv('ENVIRONMENT', 'unknown')
    return f'Hello from {env} environment!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### requirements.txt
```
Flask==2.3.0
```

### Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

CMD ["python", "app.py"]
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    image: my-flask-app:latest
    ports:
      - "5000:5000"
    environment:
      ENVIRONMENT: development
```

### Pasos
```bash
# 1. Crear todos los archivos
mkdir -p ~/docker-ejercicios/compose-build
cd ~/docker-ejercicios/compose-build

# Crear archivos

# 2. Build e iniciar
docker compose up --build -d

# 3. Test
curl http://localhost:5000

# 4. Rebuild después de cambios
# (modificar app.py)
docker compose up --build -d

# 5. Ver la imagen creada
docker images | grep my-flask-app
```

## Ejercicio 5: Múltiples Redes

### Objetivo
Separar servicios en diferentes redes.

### docker-compose.yml
```yaml
version: '3.8'

services:
  frontend:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend_net

  api:
    image: node:18-alpine
    command: sleep infinity
    networks:
      - frontend_net
      - backend_net

  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
    networks:
      - backend_net

networks:
  frontend_net:
  backend_net:
```

### Pasos
```bash
# 1. Iniciar
docker compose up -d

# 2. Verificar que frontend puede conectar a api
docker compose exec frontend ping -c 2 api

# 3. Verificar que frontend NO puede conectar a database
docker compose exec frontend ping -c 2 database
# Error

# 4. Verificar que api puede conectar a database
docker compose exec api ping -c 2 database

# 5. Inspeccionar redes
docker network ls | grep compose
docker network inspect <nombre-frontend-net>
```

## Ejercicio 6: Escalado de Servicios

### Objetivo
Escalar servicios horizontalmente.

### docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080-8085:80"

  worker:
    image: alpine
    command: sh -c 'while true; do echo "Worker $$HOSTNAME working"; sleep 5; done'
```

### Pasos
```bash
# 1. Iniciar con escala
docker compose up -d --scale worker=3 --scale web=2

# 2. Ver réplicas
docker compose ps

# 3. Ver logs de todos los workers
docker compose logs -f worker

# 4. Escalar dinámicamente
docker compose up -d --scale worker=5

# 5. Reducir escala
docker compose up -d --scale worker=2
```

## Ejercicio 7: Health Checks

### Objetivo
Configurar health checks para servicios.

### docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  api:
    image: hashicorp/http-echo
    command: ["-text=Hello from API"]
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:5678"]
      interval: 10s
      timeout: 5s
      retries: 3
```

### Pasos
```bash
# 1. Iniciar
docker compose up -d

# 2. Ver estado de salud
docker compose ps
# Verás (healthy) o (unhealthy)

# 3. Inspeccionar health
docker inspect <container-id> | grep -A 20 Health
```

## Ejercicio 8: Profiles (Entornos Dev/Prod)

### Objetivo
Diferentes configuraciones para desarrollo y producción.

### docker-compose.yml
```yaml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret

  app:
    image: myapp:latest
    depends_on:
      - db

  # Solo en desarrollo
  adminer:
    image: adminer
    ports:
      - "8080:8080"
    profiles:
      - dev

  # Solo en desarrollo
  mailhog:
    image: mailhog/mailhog
    ports:
      - "8025:8025"
    profiles:
      - dev
```

### Pasos
```bash
# 1. Producción (sin profiles)
docker compose up -d
# Solo inicia db y app

# 2. Desarrollo (con profile dev)
docker compose --profile dev up -d
# Inicia db, app, adminer y mailhog

# 3. Ver qué servicios están activos
docker compose ps
```

## Ejercicio 9: Override Files

### Objetivo
Sobrescribir configuración base con archivos específicos.

### docker-compose.yml (base)
```yaml
version: '3.8'

services:
  app:
    image: node:18-alpine
    command: npm start
```

### docker-compose.override.yml (desarrollo - automático)
```yaml
version: '3.8'

services:
  app:
    volumes:
      - ./src:/app/src
    environment:
      NODE_ENV: development
    command: npm run dev
```

### docker-compose.prod.yml (producción - manual)
```yaml
version: '3.8'

services:
  app:
    environment:
      NODE_ENV: production
    restart: unless-stopped
```

### Pasos
```bash
# 1. Desarrollo (usa override automáticamente)
docker compose up -d

# 2. Producción (especificar archivo)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 3. Ver configuración final
docker compose config
docker compose -f docker-compose.yml -f docker-compose.prod.yml config
```

## Ejercicio 10: Aplicación Completa - Stack MERN

### Objetivo
MongoDB + Express + React + Node.js

### Estructura
```
mern-app/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   └── ...
└── frontend/
    ├── Dockerfile
    └── ...
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    volumes:
      - mongo_data:/data/db
    networks:
      - backend

  backend:
    build: ./backend
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      MONGO_URL: mongodb://admin:secret@mongodb:27017
      NODE_ENV: development
    depends_on:
      - mongodb
    networks:
      - backend
      - frontend

  frontend:
    build: ./frontend
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      REACT_APP_API_URL: http://localhost:5000
    depends_on:
      - backend
    networks:
      - frontend

networks:
  frontend:
  backend:

volumes:
  mongo_data:
```

### Comandos del Proyecto
```bash
# Build todo
docker compose build

# Iniciar
docker compose up -d

# Ver logs de un servicio específico
docker compose logs -f backend

# Ejecutar comandos en servicio
docker compose exec backend npm run migrate

# Rebuild solo un servicio
docker compose up -d --build frontend

# Detener todo
docker compose down

# Eliminar todo incluyendo volúmenes
docker compose down -v
```

## Comandos Avanzados de Compose

```bash
# Ver configuración procesada
docker compose config

# Validar docker-compose.yml
docker compose config --quiet

# Listar servicios
docker compose ps

# Ejecutar comando en servicio
docker compose exec <servicio> <comando>

# Ver logs
docker compose logs
docker compose logs -f <servicio>
docker compose logs --tail=100 <servicio>

# Reiniciar servicio
docker compose restart <servicio>

# Detener sin eliminar
docker compose stop

# Eliminar solo contenedores (mantiene volúmenes e imágenes)
docker compose down

# Eliminar todo
docker compose down -v --rmi all

# Ver uso de recursos
docker compose top

# Pull de todas las imágenes
docker compose pull

# Build sin cache
docker compose build --no-cache

# Escalar servicios
docker compose up -d --scale <servicio>=<N>
```

## Mejores Prácticas

1. ✅ Usa **version específica** (3.8, no 3)
2. ✅ Define **depends_on** para orden de inicio
3. ✅ Usa **health checks** cuando sea crítico
4. ✅ Externaliza **secretos** en .env
5. ✅ Usa **volúmenes nombrados** para datos
6. ✅ Define **restart policies** apropiadas
7. ✅ Separa **redes** por capas
8. ✅ Usa **profiles** para diferentes entornos
9. ✅ Documenta con **comentarios** en YAML
10. ✅ Versionamiento de **imágenes** (no latest)

## Troubleshooting

### Problema: Servicio no inicia
```bash
# Ver logs
docker compose logs <servicio>

# Ver qué falló
docker compose ps

# Reiniciar
docker compose restart <servicio>
```

### Problema: Cambios no se reflejan
```bash
# Rebuild
docker compose up --build -d

# Forzar recreación
docker compose up -d --force-recreate
```

### Problema: Conflicto de puertos
```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "8081:80"  # En lugar de 8080:80
```

### Problema: Volúmenes con datos viejos
```bash
# Eliminar volúmenes
docker compose down -v

# Recrear desde cero
docker compose up -d
```

## Recursos Adicionales

- [Documentación oficial Compose](https://docs.docker.com/compose/)
- [Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Awesome Compose](https://github.com/docker/awesome-compose) - Ejemplos
