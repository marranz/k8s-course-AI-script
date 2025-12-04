# Proyecto Final: E-Commerce Platform

## Descripción del Proyecto

Construirás una plataforma de e-commerce completa usando Docker y Docker Compose con:
- **Frontend:** React (servidor nginx)
- **Backend API:** Node.js + Express
- **Base de datos:** PostgreSQL
- **Cache:** Redis
- **Reverse Proxy:** Nginx

**Duración estimada:** 2-3 horas

## Arquitectura del Sistema

```
                    Internet
                        │
                        ▼
                ┌───────────────┐
                │ Nginx Proxy   │ :80
                │ (reverse      │
                │  proxy)       │
                └───────┬───────┘
                        │
            ┌───────────┴──────────┐
            │                      │
     ┌──────▼──────┐      ┌───────▼────────┐
     │  Frontend   │      │   Backend API   │ :5000
     │  (React)    │      │   (Node.js)     │
     └─────────────┘      └───────┬─────────┘
                                  │
                          ┌───────┴────────┐
                          │                │
                   ┌──────▼──────┐  ┌─────▼──────┐
                   │ PostgreSQL  │  │   Redis    │
                   │  :5432      │  │   :6379    │
                   └─────────────┘  └────────────┘
```

## Estructura del Proyecto

```
ecommerce-platform/
├── docker-compose.yml
├── .env
├── .dockerignore
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── public/
│   │   └── index.html
│   └── src/
│       ├── App.js
│       ├── index.js
│       └── api.js
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── server.js
│   │   ├── db.js
│   │   ├── cache.js
│   │   └── routes/
│   │       └── products.js
│   └── .dockerignore
└── README.md
```

## Paso 1: Configuración Inicial

### Crear Estructura de Directorios

```bash
mkdir -p ecommerce-platform/{frontend/{src,public},backend/src/routes,nginx}
cd ecommerce-platform
```

### Crear archivo .env

```env
# Base de Datos
POSTGRES_USER=ecommerce_user
POSTGRES_PASSWORD=secure_password_123
POSTGRES_DB=ecommerce_db

# Backend
NODE_ENV=production
PORT=5000
DB_HOST=postgres
DB_PORT=5432
REDIS_HOST=redis
REDIS_PORT=6379

# Frontend
REACT_APP_API_URL=http://localhost/api
```

### Crear .dockerignore global

```.dockerignore
node_modules
npm-debug.log
.git
.env
.DS_Store
*.md
.vscode
```

## Paso 2: Backend (Node.js API)

### backend/package.json

```json
{
  "name": "ecommerce-backend",
  "version": "1.0.0",
  "description": "E-commerce Backend API",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0",
    "redis": "^4.6.7",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3"
  }
}
```

### backend/src/server.js

```javascript
const express = require('express');
const cors = require('cors');
const { testDBConnection, initDB } = require('./db');
const { testRedisConnection } = require('./cache');
const productsRouter = require('./routes/products');

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(express.json());

// Health check
app.get('/health', async (req, res) => {
  const dbStatus = await testDBConnection();
  const redisStatus = await testRedisConnection();

  res.json({
    status: 'OK',
    database: dbStatus ? 'connected' : 'disconnected',
    cache: redisStatus ? 'connected' : 'disconnected',
    timestamp: new Date().toISOString()
  });
});

// Routes
app.use('/api/products', productsRouter);

// Root
app.get('/', (req, res) => {
  res.json({ message: 'E-Commerce API v1.0' });
});

// Initialize and start
async function start() {
  try {
    // Wait for DB to be ready
    await new Promise(resolve => setTimeout(resolve, 5000));

    // Initialize database
    await initDB();
    console.log('✅ Database initialized');

    // Start server
    app.listen(PORT, '0.0.0.0', () => {
      console.log(`🚀 Server running on port ${PORT}`);
    });
  } catch (error) {
    console.error('❌ Failed to start server:', error);
    process.exit(1);
  }
}

start();
```

### backend/src/db.js

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  user: process.env.POSTGRES_USER,
  host: process.env.DB_HOST,
  database: process.env.POSTGRES_DB,
  password: process.env.POSTGRES_PASSWORD,
  port: process.env.DB_PORT || 5432,
});

async function testDBConnection() {
  try {
    await pool.query('SELECT NOW()');
    return true;
  } catch (error) {
    console.error('DB connection error:', error);
    return false;
  }
}

async function initDB() {
  const createTableQuery = `
    CREATE TABLE IF NOT EXISTS products (
      id SERIAL PRIMARY KEY,
      name VARCHAR(255) NOT NULL,
      description TEXT,
      price DECIMAL(10, 2) NOT NULL,
      stock INTEGER DEFAULT 0,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    );
  `;

  const insertSampleData = `
    INSERT INTO products (name, description, price, stock)
    VALUES
      ('Laptop Pro', 'High-performance laptop', 1299.99, 15),
      ('Wireless Mouse', 'Ergonomic wireless mouse', 29.99, 50),
      ('Mechanical Keyboard', 'RGB mechanical keyboard', 89.99, 30),
      ('USB-C Hub', '7-in-1 USB-C hub', 49.99, 40),
      ('Monitor 27"', '4K Ultra HD monitor', 399.99, 20)
    ON CONFLICT DO NOTHING;
  `;

  try {
    await pool.query(createTableQuery);
    await pool.query(insertSampleData);
    console.log('Database tables created and populated');
  } catch (error) {
    console.error('Error initializing database:', error);
    throw error;
  }
}

async function query(text, params) {
  return pool.query(text, params);
}

module.exports = { query, testDBConnection, initDB };
```

### backend/src/cache.js

```javascript
const redis = require('redis');

const client = redis.createClient({
  url: `redis://${process.env.REDIS_HOST}:${process.env.REDIS_PORT}`
});

client.on('error', (err) => console.error('Redis Client Error', err));
client.on('connect', () => console.log('✅ Redis connected'));

client.connect();

async function testRedisConnection() {
  try {
    await client.ping();
    return true;
  } catch (error) {
    return false;
  }
}

async function get(key) {
  try {
    return await client.get(key);
  } catch (error) {
    console.error('Redis GET error:', error);
    return null;
  }
}

async function set(key, value, expireSeconds = 3600) {
  try {
    await client.setEx(key, expireSeconds, value);
  } catch (error) {
    console.error('Redis SET error:', error);
  }
}

module.exports = { get, set, testRedisConnection };
```

### backend/src/routes/products.js

```javascript
const express = require('express');
const router = express.Router();
const db = require('../db');
const cache = require('../cache');

// Get all products
router.get('/', async (req, res) => {
  try {
    // Try cache first
    const cached = await cache.get('products:all');
    if (cached) {
      return res.json({ source: 'cache', data: JSON.parse(cached) });
    }

    // Query database
    const result = await db.query('SELECT * FROM products ORDER BY id');

    // Store in cache
    await cache.set('products:all', JSON.stringify(result.rows), 300);

    res.json({ source: 'database', data: result.rows });
  } catch (error) {
    console.error('Error fetching products:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Get product by ID
router.get('/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const result = await db.query('SELECT * FROM products WHERE id = $1', [id]);

    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'Product not found' });
    }

    res.json(result.rows[0]);
  } catch (error) {
    console.error('Error fetching product:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

module.exports = router;
```

### backend/Dockerfile

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Production image
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy application code
COPY --chown=nodejs:nodejs package*.json ./
COPY --chown=nodejs:nodejs src ./src

USER nodejs

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node -e "require('http').get('http://localhost:5000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["npm", "start"]
```

## Paso 3: Frontend (React)

### frontend/package.json

```json
{
  "name": "ecommerce-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  },
  "browserslist": {
    "production": [">0.2%", "not dead"],
    "development": ["last 1 chrome version"]
  },
  "devDependencies": {
    "react-scripts": "5.0.1"
  }
}
```

### frontend/public/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>E-Commerce Platform</title>
  <style>
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
      background: #f5f5f5;
    }
  </style>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

### frontend/src/index.js

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

### frontend/src/api.js

```javascript
const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000';

export async function fetchProducts() {
  const response = await fetch(`${API_URL}/products`);
  if (!response.ok) {
    throw new Error('Failed to fetch products');
  }
  return response.json();
}

export async function checkHealth() {
  const response = await fetch(`${API_URL}/health`);
  return response.json();
}
```

### frontend/src/App.js

```javascript
import React, { useState, useEffect } from 'react';
import { fetchProducts, checkHealth } from './api';

function App() {
  const [products, setProducts] = useState([]);
  const [health, setHealth] = useState(null);
  const [loading, setLoading] = useState(true);
  const [source, setSource] = useState('');

  useEffect(() => {
    loadData();
  }, []);

  async function loadData() {
    try {
      const [productsData, healthData] = await Promise.all([
        fetchProducts(),
        checkHealth()
      ]);
      setProducts(productsData.data);
      setSource(productsData.source);
      setHealth(healthData);
    } catch (error) {
      console.error('Error loading data:', error);
    } finally {
      setLoading(false);
    }
  }

  if (loading) {
    return <div style={styles.container}><h2>Loading...</h2></div>;
  }

  return (
    <div style={styles.container}>
      <header style={styles.header}>
        <h1>🛒 E-Commerce Platform</h1>
        <div style={styles.health}>
          {health && (
            <>
              <span>DB: {health.database === 'connected' ? '✅' : '❌'}</span>
              <span>Cache: {health.cache === 'connected' ? '✅' : '❌'}</span>
              <span>Source: {source}</span>
            </>
          )}
        </div>
      </header>

      <main style={styles.main}>
        <h2>Products</h2>
        <div style={styles.grid}>
          {products.map(product => (
            <div key={product.id} style={styles.card}>
              <h3>{product.name}</h3>
              <p>{product.description}</p>
              <div style={styles.price}>${product.price}</div>
              <div style={styles.stock}>Stock: {product.stock}</div>
            </div>
          ))}
        </div>
      </main>
    </div>
  );
}

const styles = {
  container: {
    maxWidth: '1200px',
    margin: '0 auto',
    padding: '20px',
  },
  header: {
    background: 'white',
    padding: '20px',
    borderRadius: '8px',
    marginBottom: '20px',
    boxShadow: '0 2px 4px rgba(0,0,0,0.1)',
  },
  health: {
    display: 'flex',
    gap: '15px',
    fontSize: '14px',
    marginTop: '10px',
  },
  main: {
    background: 'white',
    padding: '20px',
    borderRadius: '8px',
  },
  grid: {
    display: 'grid',
    gridTemplateColumns: 'repeat(auto-fill, minmax(250px, 1fr))',
    gap: '20px',
    marginTop: '20px',
  },
  card: {
    border: '1px solid #e0e0e0',
    borderRadius: '8px',
    padding: '15px',
    transition: 'transform 0.2s',
  },
  price: {
    fontSize: '24px',
    fontWeight: 'bold',
    color: '#2196F3',
    marginTop: '10px',
  },
  stock: {
    fontSize: '14px',
    color: '#666',
    marginTop: '5px',
  },
};

export default App;
```

### frontend/Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY public ./public
COPY src ./src

RUN npm run build

# Production stage
FROM nginx:alpine

# Copy build files
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### frontend/nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Enable gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

## Paso 4: Nginx Reverse Proxy

### nginx/nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:5000;
    }

    upstream frontend {
        server frontend:80;
    }

    server {
        listen 80;
        server_name localhost;

        # Frontend
        location / {
            proxy_pass http://frontend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Backend API
        location /api {
            rewrite ^/api/(.*) /$1 break;
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        # Health check
        location /health {
            proxy_pass http://backend/health;
        }
    }
}
```

### nginx/Dockerfile

```dockerfile
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## Paso 5: Docker Compose

### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    image: ecommerce-backend:latest
    restart: unless-stopped
    environment:
      NODE_ENV: ${NODE_ENV}
      PORT: ${PORT}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
      DB_HOST: ${DB_HOST}
      DB_PORT: ${DB_PORT}
      REDIS_HOST: ${REDIS_HOST}
      REDIS_PORT: ${REDIS_PORT}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
      - frontend

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        REACT_APP_API_URL: ${REACT_APP_API_URL}
    image: ecommerce-frontend:latest
    restart: unless-stopped
    depends_on:
      - backend
    networks:
      - frontend

  proxy:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    image: ecommerce-proxy:latest
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - frontend
      - backend
    networks:
      - frontend

networks:
  frontend:
  backend:

volumes:
  postgres_data:
  redis_data:
```

## Paso 6: Ejecutar el Proyecto

### Build e Inicio

```bash
# 1. Build de todas las imágenes
docker compose build

# 2. Iniciar todos los servicios
docker compose up -d

# 3. Ver logs
docker compose logs -f

# 4. Verificar estado
docker compose ps
```

### Acceder a la Aplicación

- **Frontend:** http://localhost
- **API directa:** http://localhost/api/products
- **Health check:** http://localhost/health

### Verificaciones

```bash
# Ver logs de un servicio específico
docker compose logs -f backend

# Ejecutar comandos en contenedores
docker compose exec backend npm list
docker compose exec postgres psql -U ecommerce_user -d ecommerce_db

# Ver uso de recursos
docker compose top

# Verificar redes
docker network ls | grep ecommerce

# Verificar volúmenes
docker volume ls | grep ecommerce
```

## Paso 7: Testing

### Test de Funcionalidad

```bash
# 1. Health check
curl http://localhost/health

# 2. Listar productos
curl http://localhost/api/products

# 3. Producto específico
curl http://localhost/api/products/1

# 4. Verificar cache (llamar 2 veces)
curl http://localhost/api/products  # source: database
curl http://localhost/api/products  # source: cache
```

### Test de Resiliencia

```bash
# Simular fallo del backend
docker compose stop backend

# Verificar que proxy maneja el error
curl http://localhost/api/products

# Recuperar
docker compose start backend
```

## Paso 8: Limpieza

```bash
# Detener servicios
docker compose down

# Eliminar también volúmenes
docker compose down -v

# Eliminar también imágenes
docker compose down -v --rmi all
```

## Criterios de Evaluación

### Funcionalidad (40%)
- [ ] Todos los servicios inician correctamente
- [ ] Frontend muestra lista de productos
- [ ] API responde correctamente
- [ ] Cache funciona (verificar source: cache)
- [ ] Health checks pasan

### Dockerfiles (20%)
- [ ] Multi-stage builds implementados
- [ ] Usuario no-root configurado
- [ ] Health checks definidos
- [ ] Imágenes optimizadas (< 100MB frontend, < 150MB backend)

### Docker Compose (20%)
- [ ] Servicios con depends_on correcto
- [ ] Redes separadas (frontend/backend)
- [ ] Volúmenes para persistencia
- [ ] Variables de entorno en .env
- [ ] Restart policies configuradas

### Seguridad (10%)
- [ ] No se exponen puertos innecesarios
- [ ] Passwords en variables de entorno
- [ ] Contenedores no ejecutan como root

### Documentación (10%)
- [ ] README con instrucciones
- [ ] Comentarios en archivos complejos
- [ ] Arquitectura documentada

## Mejoras Opcionales

### Nivel Intermedio
1. Añadir endpoint POST para crear productos
2. Implementar paginación en el listado
3. Añadir logs estructurados (Winston/Pino)
4. Configurar HTTPS en nginx

### Nivel Avanzado
1. Implementar CI/CD con GitHub Actions
2. Añadir monitoring con Prometheus + Grafana
3. Implementar autenticación JWT
4. Añadir tests unitarios y e2e
5. Deploy a Kubernetes

## Recursos

- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Compose Best Practices](https://docs.docker.com/compose/production/)
- [React Production Build](https://create-react-app.dev/docs/production-build/)
- [Express Production Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html)

## Troubleshooting Común

### Problema: Frontend no conecta al backend
**Solución:** Verificar REACT_APP_API_URL en .env

### Problema: Base de datos no inicia
**Solución:**
```bash
docker compose logs postgres
docker volume rm ecommerce-platform_postgres_data
docker compose up -d
```

### Problema: Cache no funciona
**Solución:**
```bash
docker compose exec redis redis-cli PING
docker compose logs redis
```

¡Felicidades por completar el proyecto final! 🎉
