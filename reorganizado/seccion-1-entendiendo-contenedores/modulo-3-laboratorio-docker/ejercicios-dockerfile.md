# Ejercicios de Dockerfile

## Ejercicio 1: Dockerfile Básico para Node.js

### Objetivo
Crear un Dockerfile para una aplicación Node.js simple.

### Archivos del Proyecto
**app.js**
```javascript
const http = require('http');
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hola desde Docker!\n');
});

server.listen(port, () => {
  console.log(`Server running on port ${port}`);
});
```

**package.json**
```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
```

### Tarea
Crea un Dockerfile que:
1. Use la imagen oficial de Node.js (versión 18-alpine)
2. Establezca el directorio de trabajo en `/app`
3. Copie `package.json` y `app.js`
4. Exponga el puerto 3000
5. Ejecute `node app.js`

### Comandos
```bash
# Build
docker build -t node-app:1.0 .

# Run
docker run -d -p 3000:3000 --name mi-node-app node-app:1.0

# Test
curl http://localhost:3000
```

## Ejercicio 2: Multi-Stage Build

### Objetivo
Crear una imagen optimizada usando multi-stage builds.

### Tarea
Dockerfile que:
- Stage 1 (builder): Instala todas las dependencias y compila
- Stage 2 (production): Solo copia los archivos necesarios

```dockerfile
# Tu solución aquí
```

## Ejercicio 3: Usuario No Root

### Objetivo
Mejorar la seguridad ejecutando como usuario no privilegiado.

### Tarea
Modifica el Dockerfile del Ejercicio 1:
1. Crea un grupo `appgroup` (GID 1001)
2. Crea un usuario `appuser` (UID 1001)
3. Cambia ownership de archivos
4. Ejecuta como `appuser`

## Ejercicio 4: Health Check

### Objetivo
Añadir health check a una aplicación.

### Tarea
Dockerfile con HEALTHCHECK que:
- Verifica cada 30 segundos
- Timeout de 3 segundos
- Hace curl al endpoint `/health`

## Ejercicio 5: Build Arguments

### Objetivo
Usar ARG para versiones parametrizables.

### Tarea
Dockerfile que acepta:
- `NODE_VERSION` (default: 18)
- `PORT` (default: 3000)

Build con:
```bash
docker build --build-arg NODE_VERSION=20 --build-arg PORT=8080 -t node-app:custom .
```

## Ejercicio 6: .dockerignore

### Objetivo
Optimizar build excluyendo archivos innecesarios.

### Tarea
Crea `.dockerignore` que excluya:
- node_modules
- .git
- *.log
- .env

## Ejercicio 7: Python Flask App

### Objetivo
Dockerizar una aplicación Python.

**app.py**
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello from Flask!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**requirements.txt**
```
Flask==2.3.0
```

### Tarea
Crea Dockerfile que:
1. Use python:3.11-slim
2. Instale requirements
3. Ejecute como usuario no root
4. Exponga puerto 5000

## Ejercicio 8: Go Application (Imagen Mínima)

### Objetivo
Crear la imagen más pequeña posible.

**main.go**
```go
package main
import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from Go!")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

### Tarea
Multi-stage Dockerfile:
- Stage 1: golang para compilar
- Stage 2: scratch o alpine (< 20MB)

## Proyecto Final: Aplicación Real

### Objetivo
Dockerizar una aplicación React + Node.js API.

### Estructura
```
proyecto/
├── frontend/    (React app)
├── backend/     (Express API)
└── Dockerfile   (multi-stage)
```

### Requisitos
1. Build React en stage 1
2. Build backend en stage 2
3. Nginx sirve frontend + proxy al backend
4. Imagen final < 100MB
5. Usuario no root
6. Health checks

## Soluciones

### Ejercicio 1: Solución
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json app.js ./
EXPOSE 3000
CMD ["node", "app.js"]
```

### Ejercicio 3: Solución
```dockerfile
FROM node:18-alpine

# Crear usuario
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /app
COPY --chown=appuser:appgroup package.json app.js ./

USER appuser

EXPOSE 3000
CMD ["node", "app.js"]
```

### Ejercicio 7: Solución
```dockerfile
FROM python:3.11-slim

RUN useradd -m -u 1001 appuser

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=appuser:appuser app.py .

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```
