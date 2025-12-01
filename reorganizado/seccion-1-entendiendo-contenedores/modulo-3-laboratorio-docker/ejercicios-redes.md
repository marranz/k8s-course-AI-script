# Ejercicios de Redes en Docker

## Ejercicio 1: Red Bridge por Defecto

### Objetivo
Entender cómo funciona la red bridge predeterminada.

### Pasos

```bash
# 1. Crear dos contenedores en la red por defecto
docker run -d --name contenedor1 alpine sleep 3600
docker run -d --name contenedor2 alpine sleep 3600

# 2. Inspeccionar la red bridge
docker network inspect bridge

# 3. Obtener IP de contenedor1
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' contenedor1

# 4. Desde contenedor2, hacer ping a contenedor1 (por IP)
docker exec contenedor2 ping -c 3 <IP_de_contenedor1>

# 5. Intentar ping por nombre (NO funcionará en bridge default)
docker exec contenedor2 ping -c 3 contenedor1
# Error: bad address 'contenedor1'
```

### Preguntas
- ¿Por qué funciona el ping por IP pero no por nombre?
- ¿Qué problema tiene esto para aplicaciones reales?

### Limpieza
```bash
docker rm -f contenedor1 contenedor2
```

## Ejercicio 2: Red Personalizada con DNS

### Objetivo
Usar redes personalizadas para resolución DNS automática.

### Pasos

```bash
# 1. Crear red personalizada
docker network create mi-red

# 2. Crear contenedores en esta red
docker run -d --name web --network mi-red nginx:alpine
docker run -d --name api --network mi-red alpine sleep 3600

# 3. Desde 'api', hacer ping a 'web' por NOMBRE
docker exec api ping -c 3 web
# ¡Funciona! DNS automático

# 4. Verificar resolución DNS
docker exec api nslookup web

# 5. Inspeccionar la red
docker network inspect mi-red
```

### Preguntas
- ¿Por qué ahora funciona la resolución por nombre?
- ¿Qué ventaja tiene esto?

### Limpieza
```bash
docker rm -f web api
docker network rm mi-red
```

## Ejercicio 3: Port Mapping

### Objetivo
Exponer servicios de contenedores al host.

### Pasos

```bash
# 1. Nginx sin mapeo de puerto
docker run -d --name nginx-privado nginx:alpine

# 2. Intentar acceder desde el host
curl http://localhost
# Error: Connection refused

# 3. Nginx CON mapeo de puerto
docker run -d --name nginx-publico -p 8080:80 nginx:alpine

# 4. Acceder desde el host
curl http://localhost:8080
# ¡Funciona!

# 5. Múltiples puertos
docker run -d --name multi-port \
  -p 8081:80 \
  -p 8082:443 \
  nginx:alpine

# 6. Verificar puertos mapeados
docker port multi-port
```

### Limpieza
```bash
docker rm -f nginx-privado nginx-publico multi-port
```

## Ejercicio 4: Arquitectura Multi-Tier

### Objetivo
Simular arquitectura de 3 capas con redes separadas.

### Pasos

```bash
# 1. Crear redes separadas
docker network create frontend-net
docker network create backend-net

# 2. Base de datos (solo en backend)
docker run -d \
  --name database \
  --network backend-net \
  -e POSTGRES_PASSWORD=secreto \
  postgres:15-alpine

# 3. API (en ambas redes)
docker run -d \
  --name api \
  --network backend-net \
  alpine sleep 3600

docker network connect frontend-net api

# 4. Web (solo en frontend)
docker run -d \
  --name web \
  --network frontend-net \
  -p 8080:80 \
  nginx:alpine

# 5. Verificar conectividad

# Web puede conectar a API
docker exec web ping -c 2 api

# API puede conectar a database
docker exec api ping -c 2 database

# Web NO puede conectar directamente a database
docker exec web ping -c 2 database
# Error: bad address
```

### Diagrama
```
┌─────────┐
│   WEB   │ :8080 (puerto expuesto)
└────┬────┘
     │ frontend-net
┌────▼────┐
│   API   │
└────┬────┘
     │ backend-net
┌────▼────┐
│   DB    │
└─────────┘
```

### Limpieza
```bash
docker rm -f database api web
docker network rm frontend-net backend-net
```

## Ejercicio 5: Comunicación entre Contenedores

### Objetivo
Aplicación real: API que se conecta a base de datos.

### Pasos

```bash
# 1. Crear red
docker network create app-network

# 2. Redis
docker run -d \
  --name redis \
  --network app-network \
  redis:alpine

# 3. Aplicación Node.js que usa Redis
cat > app.js << 'EOF'
const express = require('express');
const redis = require('redis');
const app = express();

const client = redis.createClient({
  url: 'redis://redis:6379'
});

client.connect();

app.get('/', async (req, res) => {
  const visits = await client.get('visits');
  const newVisits = visits ? parseInt(visits) + 1 : 1;
  await client.set('visits', newVisits);
  res.send(`Visitas: ${newVisits}`);
});

app.listen(3000, () => {
  console.log('Servidor en puerto 3000');
});
EOF

cat > package.json << 'EOF'
{
  "name": "app-redis",
  "dependencies": {
    "express": "^4.18.0",
    "redis": "^4.6.0"
  }
}
EOF

cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY app.js .
CMD ["node", "app.js"]
EOF

# 4. Build y run
docker build -t app-redis .

docker run -d \
  --name app \
  --network app-network \
  -p 3000:3000 \
  app-redis

# 5. Probar la aplicación
sleep 5
curl http://localhost:3000  # Visitas: 1
curl http://localhost:3000  # Visitas: 2
curl http://localhost:3000  # Visitas: 3
```

### Limpieza
```bash
docker rm -f redis app
docker network rm app-network
docker rmi app-redis
rm app.js package.json Dockerfile
```

## Ejercicio 6: Red Host

### Objetivo
Entender cuándo usar red tipo host.

### Pasos

```bash
# 1. Contenedor con red bridge (default)
docker run -d --name nginx-bridge -p 8080:80 nginx:alpine

# 2. Contenedor con red host
docker run -d --name nginx-host --network host nginx:alpine

# 3. Acceder a ambos
curl http://localhost:8080  # nginx-bridge
curl http://localhost:80    # nginx-host (puerto directo)

# 4. Inspeccionar diferencias
docker inspect nginx-bridge | grep IPAddress
docker inspect nginx-host | grep IPAddress
# nginx-host no tiene IP propia
```

### Preguntas
- ¿Cuándo usarías red host?
- ¿Qué desventajas tiene?

### Limpieza
```bash
docker rm -f nginx-bridge nginx-host
```

## Ejercicio 7: Red None (Aislamiento Completo)

### Objetivo
Contenedor sin conectividad de red.

### Pasos

```bash
# 1. Crear contenedor aislado
docker run -d --name aislado --network none alpine sleep 3600

# 2. Intentar ping a internet
docker exec aislado ping -c 2 8.8.8.8
# Error: Network unreachable

# 3. Inspeccionar interfaces de red
docker exec aislado ip addr
# Solo loopback (lo)
```

### Caso de Uso
- Procesamiento de datos sensibles
- Contenedores que solo procesan archivos locales

### Limpieza
```bash
docker rm -f aislado
```

## Ejercicio 8: DNS Personalizado

### Objetivo
Configurar servidores DNS custom.

### Pasos

```bash
# 1. Contenedor con DNS específico
docker run -d \
  --name web-dns \
  --dns 8.8.8.8 \
  --dns 1.1.1.1 \
  alpine sleep 3600

# 2. Verificar DNS
docker exec web-dns cat /etc/resolv.conf

# 3. Test de resolución
docker exec web-dns nslookup google.com
```

### Limpieza
```bash
docker rm -f web-dns
```

## Ejercicio 9: Proxy Reverso con Nginx

### Objetivo
Nginx como proxy hacia múltiples backends.

### Pasos

```bash
# 1. Crear red
docker network create proxy-net

# 2. Dos servidores backend
docker run -d --name backend1 --network proxy-net \
  -e MESSAGE="Backend 1" \
  hashicorp/http-echo -text="Hello from Backend 1"

docker run -d --name backend2 --network proxy-net \
  -e MESSAGE="Backend 2" \
  hashicorp/http-echo -text="Hello from Backend 2"

# 3. Configuración de Nginx
mkdir -p ~/docker-ejercicios/nginx-proxy

cat > ~/docker-ejercicios/nginx-proxy/nginx.conf << 'EOF'
events {}

http {
    upstream backend {
        server backend1:5678;
        server backend2:5678;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
EOF

# 4. Nginx como proxy
docker run -d \
  --name proxy \
  --network proxy-net \
  -p 8080:80 \
  -v ~/docker-ejercicios/nginx-proxy/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx:alpine

# 5. Probar load balancing
curl http://localhost:8080  # Backend 1
curl http://localhost:8080  # Backend 2
curl http://localhost:8080  # Backend 1
```

### Limpieza
```bash
docker rm -f backend1 backend2 proxy
docker network rm proxy-net
rm -rf ~/docker-ejercicios/nginx-proxy
```

## Ejercicio 10: Inspección y Debugging de Redes

### Objetivo
Herramientas para diagnosticar problemas de red.

### Pasos

```bash
# 1. Crear red y contenedores
docker network create debug-net

docker run -d --name app1 --network debug-net alpine sleep 3600
docker run -d --name app2 --network debug-net alpine sleep 3600

# 2. Listar todas las redes
docker network ls

# 3. Inspeccionar red específica
docker network inspect debug-net

# 4. Ver qué contenedores están en la red
docker network inspect debug-net | grep Name

# 5. Desde un contenedor, instalar herramientas de red
docker exec app1 apk add --no-cache curl bind-tools iputils

# 6. Diagnóstico completo
docker exec app1 ping -c 2 app2
docker exec app1 nslookup app2
docker exec app1 traceroute app2
docker exec app1 ip addr
docker exec app1 ip route

# 7. Ver puertos escuchando
docker exec app1 netstat -tlnp 2>/dev/null || echo "netstat no disponible"

# 8. Desconectar y reconectar contenedor
docker network disconnect debug-net app1
docker network connect debug-net app1
```

### Limpieza
```bash
docker rm -f app1 app2
docker network rm debug-net
```

## Proyecto Práctico: Microservicios

### Objetivo
Sistema completo con frontend, backend, cache y base de datos.

### Arquitectura
```
Internet
   │
   ▼
┌──────────────┐
│ Nginx :80    │ (frontend-net, proxy-net)
└──────┬───────┘
       │
┌──────▼───────┐
│ API :3000    │ (proxy-net, backend-net)
└──────┬───────┘
       │
   ┌───┴────┐
   ▼        ▼
┌─────┐  ┌────────┐
│Redis│  │Postgres│ (backend-net)
└─────┘  └────────┘
```

### Implementación

```bash
# 1. Crear redes
docker network create frontend-net
docker network create proxy-net
docker network create backend-net

# 2. Base de datos
docker run -d \
  --name postgres \
  --network backend-net \
  -e POSTGRES_PASSWORD=secreto \
  -e POSTGRES_DB=appdb \
  postgres:15-alpine

# 3. Cache
docker run -d \
  --name redis \
  --network backend-net \
  redis:alpine

# 4. API (conectada a dos redes)
docker run -d \
  --name api \
  --network backend-net \
  -e DB_HOST=postgres \
  -e REDIS_HOST=redis \
  myapi:latest  # (crear tu propia imagen)

docker network connect proxy-net api

# 5. Frontend
docker run -d \
  --name frontend \
  --network proxy-net \
  nginx:alpine

docker network connect frontend-net frontend

# 6. Nginx proxy
docker run -d \
  --name proxy \
  --network frontend-net \
  -p 80:80 \
  nginx-proxy:latest  # (con configuración de proxy)
```

## Comandos Útiles de Red

```bash
# Listar redes
docker network ls

# Crear red
docker network create <nombre>
docker network create --driver bridge --subnet 172.25.0.0/16 mi-red

# Inspeccionar
docker network inspect <nombre>

# Conectar/Desconectar contenedor
docker network connect <red> <contenedor>
docker network disconnect <red> <contenedor>

# Eliminar red
docker network rm <nombre>

# Limpiar redes no usadas
docker network prune

# Ver puertos de un contenedor
docker port <contenedor>
```

## Mejores Prácticas

1. ✅ Usa **redes personalizadas** (no default bridge)
2. ✅ Separa **capas** con redes diferentes (frontend/backend)
3. ✅ Usa **nombres de servicio** para conectividad (no IPs)
4. ✅ **No expongas** puertos innecesarios
5. ✅ Usa **aliases** de red cuando sea útil
6. ✅ Documenta la **arquitectura de red** de tu aplicación
7. ✅ Considera **overlay networks** para multi-host (Docker Swarm/K8s)

## Troubleshooting

### Problema: Contenedores no se pueden comunicar
```bash
# Verificar que están en la misma red
docker network inspect <red>

# Verificar DNS
docker exec <contenedor> nslookup <otro-contenedor>

# Verificar conectividad
docker exec <contenedor> ping <otro-contenedor>
```

### Problema: Puerto ya en uso
```bash
# Ver qué está usando el puerto
lsof -i :8080  # macOS/Linux
netstat -ano | findstr :8080  # Windows

# Usar otro puerto
docker run -p 8081:80 nginx
```

### Problema: Contenedor no accede a internet
```bash
# Verificar DNS
docker exec <contenedor> cat /etc/resolv.conf

# Probar conectividad
docker exec <contenedor> ping 8.8.8.8
docker exec <contenedor> ping google.com
```
