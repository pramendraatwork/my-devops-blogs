---
title: "Docker Projects — Beginner to Advanced (3 Real Projects)"
date: 2024-03-24
draft: false
description: "3 hands-on Docker projects: beginner containerize a Node.js app, intermediate multi-container app with Docker Compose, and advanced production setup with Nginx, SSL, and monitoring."
categories: ["docker"]
tags: ["docker", "containers", "projects", "devops", "compose", "nginx", "monitoring"]
showToc: true
TocOpen: true
---

## Why These Projects? 🎯

Docker theory is easy to forget. Building real projects makes it stick. These 3 projects go from containerizing your first app to running a production-grade multi-service system with monitoring.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → Containerize a Node.js REST API
Project 2 (Intermediate) → Multi-Container App with Compose
Project 3 (Advanced)     → Production Stack with Nginx + SSL + Monitoring
──────────────────────────────────────────────────────────────
```

---

## Project 1: Containerize a Node.js REST API 🟢 Beginner

### What You'll Build

Take a simple Node.js REST API and containerize it from scratch — write a Dockerfile, build the image, run it locally, push it to Docker Hub, and pull it on any machine.

```
WHAT YOU'LL DO:
──────────────────────────────────────────────────────────────
✅ Write a Dockerfile step by step
✅ Build a Docker image
✅ Run a container with port mapping
✅ Use environment variables
✅ Add a health check
✅ Push to Docker Hub
✅ Pull and run on any machine
──────────────────────────────────────────────────────────────

RESULT:
Anyone can run your app with ONE command:
docker run -p 3000:3000 pramendraatwork/my-api:v1
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- Writing Dockerfiles
- Build and run Docker images
- Port mapping and environment variables
- Docker Hub push/pull
- Health checks
- .dockerignore

### Step 1: Create the Application

```bash
# Create project
mkdir docker-first-app && cd docker-first-app

# Initialize Node.js
npm init -y
npm install express
```

```javascript
// src/app.js
const express = require('express');
const app = express();
app.use(express.json());

// In-memory data store
let books = [
  { id: 1, title: 'The DevOps Handbook', author: 'Gene Kim', year: 2016 },
  { id: 2, title: 'Site Reliability Engineering', author: 'Google', year: 2016 },
  { id: 3, title: 'Docker Deep Dive', author: 'Nigel Poulton', year: 2023 }
];
let nextId = 4;

// Routes
app.get('/', (req, res) => {
  res.json({
    message: '📚 Books API - Running in Docker!',
    version: process.env.APP_VERSION || '1.0.0',
    environment: process.env.NODE_ENV || 'development',
    endpoints: ['GET /books', 'POST /books', 'GET /books/:id', 'DELETE /books/:id']
  });
});

app.get('/books', (req, res) => {
  res.json({ total: books.length, books });
});

app.get('/books/:id', (req, res) => {
  const book = books.find(b => b.id === parseInt(req.params.id));
  if (!book) return res.status(404).json({ error: 'Book not found' });
  res.json(book);
});

app.post('/books', (req, res) => {
  const { title, author, year } = req.body;
  if (!title || !author) {
    return res.status(400).json({ error: 'Title and author are required' });
  }
  const book = { id: nextId++, title, author, year: year || new Date().getFullYear() };
  books.push(book);
  res.status(201).json(book);
});

app.delete('/books/:id', (req, res) => {
  const index = books.findIndex(b => b.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ error: 'Book not found' });
  books.splice(index, 1);
  res.json({ message: 'Book deleted successfully' });
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    uptime: Math.floor(process.uptime()),
    memory: process.memoryUsage().rss,
    timestamp: new Date().toISOString()
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Books API running on port ${PORT}`);
  console.log(`📦 Environment: ${process.env.NODE_ENV || 'development'}`);
  console.log(`🏷️  Version: ${process.env.APP_VERSION || '1.0.0'}`);
});
```

```json
// package.json
{
  "name": "docker-first-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/app.js",
    "dev": "nodemon src/app.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### Step 2: Create .dockerignore

```bash
# .dockerignore — files NOT copied into image
cat > .dockerignore << 'EOF'
node_modules/
npm-debug.log
.git/
.gitignore
*.md
.env
.env.*
coverage/
.nyc_output/
Dockerfile
.dockerignore
EOF
```

### Step 3: Write the Dockerfile

```dockerfile
# Dockerfile
# ── Stage 1: Base ─────────────────────────────────────────────
# Use official Node.js Alpine image (small — only ~7MB base)
FROM node:20-alpine AS base

# Set working directory inside container
WORKDIR /app

# ── Stage 2: Dependencies ─────────────────────────────────────
FROM base AS deps

# Copy ONLY package files first
# This layer is cached as long as package.json doesn't change!
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# ── Stage 3: Runner ───────────────────────────────────────────
FROM base AS runner

# Copy installed dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules

# Copy application source code
COPY src/ ./src/
COPY package.json ./

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Switch to non-root user
USER nodejs

# Document the port the app uses
EXPOSE 3000

# Health check — Docker checks this every 30s
HEALTHCHECK --interval=30s \
            --timeout=10s \
            --start-period=20s \
            --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

# Default environment variables
ENV NODE_ENV=production \
    PORT=3000 \
    APP_VERSION=1.0.0

# Start the application
CMD ["node", "src/app.js"]
```

### Step 4: Build the Image

```bash
# Build the image
docker build -t books-api:v1 .

# Check it was created
docker images | grep books-api

# See the layers
docker history books-api:v1

# Inspect the image
docker inspect books-api:v1 | grep -A 5 "Env"
```

### Step 5: Run the Container

```bash
# Basic run
docker run -p 3000:3000 books-api:v1

# Run detached (background)
docker run -d \
  --name books-api \
  -p 3000:3000 \
  books-api:v1

# Run with custom environment variables
docker run -d \
  --name books-api \
  -p 3000:3000 \
  -e NODE_ENV=development \
  -e APP_VERSION=1.0.0 \
  books-api:v1

# Check it's running
docker ps

# View logs
docker logs books-api
docker logs -f books-api    # follow logs

# Test the API
curl http://localhost:3000/
curl http://localhost:3000/books
curl http://localhost:3000/health

# Add a book
curl -X POST http://localhost:3000/books \
  -H "Content-Type: application/json" \
  -d '{"title": "Kubernetes in Action", "author": "Marko Luksa"}'

# Shell into running container
docker exec -it books-api sh
```

### Step 6: Check Health Status

```bash
# See health status in docker ps
docker ps
# CONTAINER ID  IMAGE        STATUS
# abc123        books-api:v1 Up 2 minutes (healthy)

# Detailed health info
docker inspect books-api | grep -A 15 '"Health"'

# Force health check now
docker inspect books-api --format='{{.State.Health.Status}}'
```

### Step 7: Push to Docker Hub

```bash
# Login to Docker Hub
docker login

# Tag image with your Docker Hub username
docker tag books-api:v1 pramendraatwork/books-api:v1
docker tag books-api:v1 pramendraatwork/books-api:latest

# Push
docker push pramendraatwork/books-api:v1
docker push pramendraatwork/books-api:latest

# Now ANYONE can run it!
docker run -p 3000:3000 pramendraatwork/books-api:v1
```

### Step 8: Test Different Scenarios

```bash
# Test restart behavior
docker stop books-api
docker start books-api
docker logs books-api --tail 5

# Test with restart policy
docker run -d \
  --name books-api-auto \
  --restart unless-stopped \
  -p 3001:3000 \
  books-api:v1

# Kill the process inside — Docker restarts automatically!
docker exec books-api-auto kill 1
sleep 3
docker ps   # should show it restarted!

# Resource limits
docker run -d \
  --name books-api-limited \
  --memory="128m" \
  --cpus="0.5" \
  -p 3002:3000 \
  books-api:v1

# Check resource usage
docker stats books-api-limited --no-stream

# Cleanup
docker stop books-api books-api-auto books-api-limited
docker rm books-api books-api-auto books-api-limited
```

### Project 1 Summary

```
What you built:
──────────────────────────────────────────────────────────────
📦 Multi-stage Dockerfile (build → runner)
🔒 Non-root user for security
🏥 Health check built in
🌍 Environment variable configuration
🐳 Published to Docker Hub
──────────────────────────────────────────────────────────────

Key commands learned:
docker build -t name:tag .
docker run -d -p host:container --name name image
docker logs -f name
docker exec -it name sh
docker push username/image:tag
```

---

## Project 2: Multi-Container App with Docker Compose 🟡 Intermediate

### What You'll Build

A full-stack application — Node.js API + PostgreSQL database + Redis cache + Adminer (DB UI) — all running together with Docker Compose. One command starts everything.

```
ARCHITECTURE:
──────────────────────────────────────────────────────────────

Browser
  │
  ▼
┌─────────────────────────────────────────────────┐
│              Docker Network: app-network          │
│                                                   │
│  ┌───────────┐    ┌──────────┐    ┌───────────┐  │
│  │  Node.js  │───▶│ Postgres │    │   Redis   │  │
│  │  API      │    │ :5432    │    │  :6379    │  │
│  │  :3000    │───▶└──────────┘    └───────────┘  │
│  └───────────┘                                    │
│       │                                           │
│  ┌────▼──────┐                                   │
│  │  Adminer  │  ← DB web UI at :8080             │
│  │  :8080    │                                   │
│  └───────────┘                                   │
└─────────────────────────────────────────────────┘

Volumes:
  pgdata  → PostgreSQL data (persists across restarts)
  redis   → Redis data
──────────────────────────────────────────────────────────────
```

### Project Structure

```
docker-compose-app/
├── api/
│   ├── src/
│   │   ├── app.js
│   │   ├── db.js
│   │   └── cache.js
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml
├── docker-compose.dev.yml      ← development overrides
├── docker-compose.prod.yml     ← production overrides
├── .env.example
└── .env                        ← your secrets (gitignored!)
```

### The Application

```javascript
// api/src/db.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 5432,
  database: process.env.DB_NAME || 'appdb',
  user: process.env.DB_USER || 'admin',
  password: process.env.DB_PASSWORD || 'secret',
  max: 10,
  idleTimeoutMillis: 30000
});

// Create tables on startup
async function initDB() {
  const client = await pool.connect();
  try {
    await client.query(`
      CREATE TABLE IF NOT EXISTS tasks (
        id SERIAL PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        description TEXT,
        status VARCHAR(50) DEFAULT 'pending',
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);
    console.log('✅ Database initialized');
  } finally {
    client.release();
  }
}

module.exports = { pool, initDB };
```

```javascript
// api/src/cache.js
const redis = require('redis');

const client = redis.createClient({
  url: `redis://${process.env.REDIS_HOST || 'localhost'}:${process.env.REDIS_PORT || 6379}`
});

client.on('connect', () => console.log('✅ Redis connected'));
client.on('error', (err) => console.error('❌ Redis error:', err));

async function connectRedis() {
  await client.connect();
}

async function getCache(key) {
  const data = await client.get(key);
  return data ? JSON.parse(data) : null;
}

async function setCache(key, data, ttl = 60) {
  await client.setEx(key, ttl, JSON.stringify(data));
}

async function deleteCache(key) {
  await client.del(key);
}

module.exports = { connectRedis, getCache, setCache, deleteCache };
```

```javascript
// api/src/app.js
const express = require('express');
const { pool, initDB } = require('./db');
const { connectRedis, getCache, setCache, deleteCache } = require('./cache');

const app = express();
app.use(express.json());

// Initialize connections
async function startup() {
  try {
    await connectRedis();
    await initDB();
    console.log('🚀 All services connected!');
  } catch (err) {
    console.error('❌ Startup failed:', err.message);
    process.exit(1);
  }
}

// Routes
app.get('/health', async (req, res) => {
  try {
    // Check DB connection
    await pool.query('SELECT 1');
    res.json({
      status: 'healthy',
      database: 'connected',
      cache: 'connected',
      timestamp: new Date().toISOString()
    });
  } catch (err) {
    res.status(503).json({ status: 'unhealthy', error: err.message });
  }
});

// GET all tasks (with Redis caching!)
app.get('/tasks', async (req, res) => {
  try {
    // Check cache first
    const cached = await getCache('all_tasks');
    if (cached) {
      console.log('📦 Serving from Redis cache');
      return res.json({ source: 'cache', ...cached });
    }

    // Get from database
    const result = await pool.query(
      'SELECT * FROM tasks ORDER BY created_at DESC'
    );

    const data = { total: result.rows.length, tasks: result.rows };

    // Store in cache for 60 seconds
    await setCache('all_tasks', data, 60);
    console.log('🗄️  Serving from database');

    res.json({ source: 'database', ...data });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// GET single task
app.get('/tasks/:id', async (req, res) => {
  try {
    const cacheKey = `task_${req.params.id}`;
    const cached = await getCache(cacheKey);
    if (cached) return res.json(cached);

    const result = await pool.query(
      'SELECT * FROM tasks WHERE id = $1', [req.params.id]
    );
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'Task not found' });
    }

    await setCache(cacheKey, result.rows[0], 120);
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// POST create task
app.post('/tasks', async (req, res) => {
  const { title, description } = req.body;
  if (!title) return res.status(400).json({ error: 'Title is required' });

  try {
    const result = await pool.query(
      'INSERT INTO tasks (title, description) VALUES ($1, $2) RETURNING *',
      [title, description]
    );

    // Invalidate cache
    await deleteCache('all_tasks');

    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// PUT update task status
app.put('/tasks/:id', async (req, res) => {
  const { status } = req.body;
  const validStatuses = ['pending', 'in-progress', 'done'];

  if (!validStatuses.includes(status)) {
    return res.status(400).json({
      error: `Status must be one of: ${validStatuses.join(', ')}`
    });
  }

  try {
    const result = await pool.query(
      `UPDATE tasks
       SET status = $1, updated_at = CURRENT_TIMESTAMP
       WHERE id = $2 RETURNING *`,
      [status, req.params.id]
    );

    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'Task not found' });
    }

    // Invalidate caches
    await deleteCache('all_tasks');
    await deleteCache(`task_${req.params.id}`);

    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// DELETE task
app.delete('/tasks/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'DELETE FROM tasks WHERE id = $1 RETURNING *',
      [req.params.id]
    );

    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'Task not found' });
    }

    await deleteCache('all_tasks');
    await deleteCache(`task_${req.params.id}`);

    res.json({ message: 'Task deleted', task: result.rows[0] });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Stats endpoint
app.get('/stats', async (req, res) => {
  try {
    const result = await pool.query(`
      SELECT
        COUNT(*) as total,
        COUNT(*) FILTER (WHERE status = 'pending') as pending,
        COUNT(*) FILTER (WHERE status = 'in-progress') as in_progress,
        COUNT(*) FILTER (WHERE status = 'done') as done
      FROM tasks
    `);
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, async () => {
  console.log(`🚀 API running on port ${PORT}`);
  await startup();
});
```

```dockerfile
# api/Dockerfile
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY src/ ./src/
COPY package.json ./
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "src/app.js"]
```

### Environment File

```bash
# .env.example — copy to .env and fill in values
cat > .env.example << 'EOF'
# Database
DB_HOST=postgres
DB_PORT=5432
DB_NAME=appdb
DB_USER=admin
DB_PASSWORD=changeme_in_production

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# App
NODE_ENV=production
PORT=3000
APP_VERSION=1.0.0
EOF

# Create .env from example
cp .env.example .env
# Edit .env with your values
```

### Docker Compose Files

```yaml
# docker-compose.yml — base config
version: '3.9'

services:
  # ── API ───────────────────────────────────────────────────
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: tasks-api
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - PORT=3000
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=${DB_NAME:-appdb}
      - DB_USER=${DB_USER:-admin}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - APP_VERSION=${APP_VERSION:-1.0.0}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  # ── PostgreSQL ────────────────────────────────────────────
  postgres:
    image: postgres:16-alpine
    container_name: tasks-postgres
    environment:
      POSTGRES_DB: ${DB_NAME:-appdb}
      POSTGRES_USER: ${DB_USER:-admin}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./api/sql/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${DB_USER:-admin} -d ${DB_NAME:-appdb}']
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: unless-stopped
    networks:
      - app-network

  # ── Redis ─────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: tasks-redis
    command: redis-server --appendonly yes
    volumes:
      - redisdata:/data
    healthcheck:
      test: ['CMD', 'redis-cli', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - app-network

  # ── Adminer (DB UI) ───────────────────────────────────────
  adminer:
    image: adminer:latest
    container_name: tasks-adminer
    ports:
      - '8080:8080'
    depends_on:
      - postgres
    restart: unless-stopped
    networks:
      - app-network

networks:
  app-network:
    name: tasks-network
    driver: bridge

volumes:
  pgdata:
    name: tasks-pgdata
  redisdata:
    name: tasks-redisdata
```

```yaml
# docker-compose.dev.yml — development overrides
version: '3.9'

services:
  api:
    build:
      target: runner
    ports:
      - '3000:3000'    # expose in dev
    environment:
      - NODE_ENV=development
    volumes:
      - ./api/src:/app/src:ro    # hot reload source
    command: node src/app.js

  postgres:
    ports:
      - '5432:5432'    # expose DB port for local tools

  redis:
    ports:
      - '6379:6379'    # expose Redis port for local tools
```

```yaml
# docker-compose.prod.yml — production overrides
version: '3.9'

services:
  api:
    image: pramendraatwork/tasks-api:${IMAGE_TAG:-latest}
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
        reservations:
          cpus: '0.25'
          memory: 128M
    logging:
      driver: 'json-file'
      options:
        max-size: '10m'
        max-file: '3'

  postgres:
    # Don't expose in production!
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  adminer:
    # Don't run adminer in production!
    profiles:
      - debug
```

### Commands to Run Everything

```bash
# Development — with live reload
docker compose -f docker-compose.yml \
               -f docker-compose.dev.yml \
               up --build

# Production
docker compose -f docker-compose.yml \
               -f docker-compose.prod.yml \
               up -d

# View all running services
docker compose ps

# View logs
docker compose logs -f
docker compose logs -f api       # specific service

# Scale API (run 3 instances)
docker compose up -d --scale api=3

# Execute command in service
docker compose exec api sh
docker compose exec postgres psql -U admin -d appdb

# Restart one service
docker compose restart api

# Stop everything
docker compose down

# Stop and remove volumes (careful — deletes data!)
docker compose down -v
```

### Test the API

```bash
# Health check
curl http://localhost:3000/health

# Create tasks
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Learn Docker Compose", "description": "Build multi-container apps"}'

curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Deploy to production", "description": "Use docker compose prod"}'

# Get all tasks (first call from DB, second from cache!)
curl http://localhost:3000/tasks
curl http://localhost:3000/tasks    # notice: source changes to "cache"!

# Update task status
curl -X PUT http://localhost:3000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "in-progress"}'

# Get stats
curl http://localhost:3000/stats

# Open Adminer UI
open http://localhost:8080
# System: PostgreSQL, Server: postgres, User: admin, Password: secret, DB: appdb
```

### What You Learned

- ✅ Multi-container Docker Compose
- ✅ Service dependencies with health checks
- ✅ Named volumes for data persistence
- ✅ Custom Docker networks
- ✅ Environment variables with .env
- ✅ Dev vs production compose files
- ✅ Redis caching pattern
- ✅ PostgreSQL with Docker

---

## Project 3: Production Stack with Nginx + SSL + Monitoring 🔴 Advanced

### What You'll Build

A complete production-ready Docker setup — Nginx reverse proxy with SSL termination, Node.js API cluster, PostgreSQL, Redis, Prometheus monitoring, and Grafana dashboards. This is what real production looks like.

```
PRODUCTION ARCHITECTURE:
──────────────────────────────────────────────────────────────

Internet (HTTPS)
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│                    NGINX (Port 80/443)                   │
│              SSL termination + reverse proxy             │
└──────────┬──────────────────────────┬────────────────────┘
           │                          │
     /api/* │                    /* (frontend)
           │
    ┌──────▼──────────────────────┐
    │     API Cluster (3 replicas) │
    │  api_1  │  api_2  │  api_3  │
    └──────┬──────────────────────┘
           │
    ┌──────┼──────────┐
    ▼      ▼          ▼
 Postgres  Redis  Prometheus
  (data)  (cache)  (metrics)
                       │
                   Grafana
                  (dashboards)
──────────────────────────────────────────────────────────────
```

### Project Structure

```
docker-production/
├── nginx/
│   ├── nginx.conf
│   ├── ssl/
│   │   ├── cert.pem        ← SSL certificate
│   │   └── key.pem         ← SSL private key
│   └── conf.d/
│       └── app.conf
├── api/
│   ├── src/app.js
│   └── Dockerfile
├── monitoring/
│   ├── prometheus.yml
│   └── grafana/
│       └── dashboards/
│           └── docker.json
├── docker-compose.yml
├── docker-compose.monitoring.yml
└── .env
```

### Nginx Configuration

```nginx
# nginx/nginx.conf
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" '
                    'rt=$request_time uct=$upstream_connect_time '
                    'uht=$upstream_header_time urt=$upstream_response_time';

    access_log /var/log/nginx/access.log main;

    # Performance
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

    # Security headers
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
    server_tokens off;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;

    # Upstream — load balance across API instances
    upstream api_cluster {
        least_conn;    # send to least-busy server
        server api_1:3000;
        server api_2:3000;
        server api_3:3000;
        keepalive 32;
    }

    include /etc/nginx/conf.d/*.conf;
}
```

```nginx
# nginx/conf.d/app.conf

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

# HTTPS server
server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL configuration
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # API routes
    location /api/ {
        # Rate limiting
        limit_req zone=api burst=20 nodelay;

        proxy_pass http://api_cluster/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check (no rate limiting)
    location /health {
        proxy_pass http://api_cluster/health;
        proxy_set_header Host $host;
        access_log off;
    }

    # Static files (frontend)
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Block common attack vectors
    location ~ /\. { deny all; }
    location ~ /\.git { deny all; }
    location ~* (eval\(|base64_decode) { deny all; }
}
```

### Production Docker Compose

```yaml
# docker-compose.yml
version: '3.9'

services:
  # ── Nginx Reverse Proxy ───────────────────────────────────
  nginx:
    image: nginx:1.25-alpine
    container_name: prod-nginx
    ports:
      - '80:80'
      - '443:443'
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - ./frontend/dist:/usr/share/nginx/html:ro
      - nginx_logs:/var/log/nginx
    depends_on:
      - api_1
      - api_2
      - api_3
    restart: always
    networks:
      - frontend-net
      - backend-net
    healthcheck:
      test: ['CMD', 'nginx', '-t']
      interval: 30s
      timeout: 10s
      retries: 3

  # ── API Cluster (3 instances) ─────────────────────────────
  api_1:
    &api-base
    build:
      context: ./api
      dockerfile: Dockerfile
    image: prod-api:latest
    environment:
      - NODE_ENV=production
      - PORT=3000
      - INSTANCE=1
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: always
    networks:
      - backend-net
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
    logging:
      driver: json-file
      options:
        max-size: '10m'
        max-file: '3'

  api_2:
    <<: *api-base   # inherit all settings from api_1!
    environment:
      - NODE_ENV=production
      - PORT=3000
      - INSTANCE=2
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379

  api_3:
    <<: *api-base
    environment:
      - NODE_ENV=production
      - PORT=3000
      - INSTANCE=3
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=${DB_NAME}
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - REDIS_HOST=redis
      - REDIS_PORT=6379

  # ── PostgreSQL ────────────────────────────────────────────
  postgres:
    image: postgres:16-alpine
    container_name: prod-postgres
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U ${DB_USER} -d ${DB_NAME}']
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: always
    networks:
      - backend-net
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  # ── Redis ─────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: prod-redis
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --appendonly yes
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
    volumes:
      - redisdata:/data
    healthcheck:
      test: ['CMD', 'redis-cli', '-a', '${REDIS_PASSWORD}', 'ping']
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always
    networks:
      - backend-net
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 256M

networks:
  frontend-net:
    name: prod-frontend
  backend-net:
    name: prod-backend
    internal: true    # no direct internet access!

volumes:
  pgdata:
    name: prod-pgdata
  redisdata:
    name: prod-redisdata
  nginx_logs:
    name: prod-nginx-logs
```

### Monitoring Stack

```yaml
# docker-compose.monitoring.yml
version: '3.9'

services:
  # ── cAdvisor — Container metrics ─────────────────────────
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - '8081:8080'
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    privileged: true
    restart: unless-stopped
    networks:
      - monitoring-net

  # ── Node Exporter — Host metrics ─────────────────────────
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    ports:
      - '9100:9100'
    restart: unless-stopped
    networks:
      - monitoring-net

  # ── Prometheus — Metrics collection ──────────────────────
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    ports:
      - '9090:9090'
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    restart: unless-stopped
    networks:
      - monitoring-net
      - prod-backend    # access to app metrics

  # ── Grafana — Dashboards ──────────────────────────────────
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - '3001:3000'
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-clock-panel
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
    depends_on:
      - prometheus
    restart: unless-stopped
    networks:
      - monitoring-net

networks:
  monitoring-net:
    name: monitoring

volumes:
  prometheus_data:
  grafana_data:
```

### Prometheus Config

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Container metrics
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Host metrics
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  # API metrics (if you add /metrics endpoint)
  - job_name: 'api'
    static_configs:
      - targets: ['api_1:3000', 'api_2:3000', 'api_3:3000']
    metrics_path: '/metrics'
```

### SSL Certificate Setup

```bash
# Generate self-signed cert for testing
mkdir -p nginx/ssl

openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout nginx/ssl/key.pem \
  -out nginx/ssl/cert.pem \
  -subj "/C=IN/ST=MP/L=Indore/O=DevOps/CN=yourdomain.com"

# For production — use Let's Encrypt with certbot
# certbot certonly --standalone -d yourdomain.com
# cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem nginx/ssl/cert.pem
# cp /etc/letsencrypt/live/yourdomain.com/privkey.pem nginx/ssl/key.pem
```

### Deployment Script

```bash
#!/bin/bash
# deploy.sh — Full production deployment

set -euo pipefail

echo "🚀 Starting production deployment..."

# Check .env exists
if [ ! -f .env ]; then
  echo "❌ .env file missing!"
  exit 1
fi

# Pull latest images
echo "📦 Pulling latest images..."
docker compose pull

# Build API image
echo "🔨 Building API image..."
docker compose build api_1

# Run DB migrations
echo "🗄️  Running database migrations..."
docker compose run --rm api_1 node src/migrate.js

# Deploy with rolling update
echo "🔄 Deploying services..."

# Update one API at a time
docker compose up -d postgres redis
sleep 10

docker compose up -d api_1
sleep 10

docker compose up -d api_2
sleep 10

docker compose up -d api_3
sleep 10

docker compose up -d nginx

# Wait and check health
echo "🏥 Checking health..."
sleep 15

HEALTH=$(curl -sf http://localhost/health | python3 -c "import sys,json; print(json.load(sys.stdin)['status'])" 2>/dev/null || echo "failed")

if [ "$HEALTH" != "healthy" ]; then
  echo "❌ Health check failed! Rolling back..."
  docker compose down
  exit 1
fi

echo "✅ Deployment successful!"
echo ""
echo "📊 Service status:"
docker compose ps

echo ""
echo "🌐 App:       https://yourdomain.com"
echo "📈 Grafana:   http://localhost:3001"
echo "🔥 Prometheus: http://localhost:9090"
```

### Start Everything

```bash
# Create .env
cat > .env << 'EOF'
DB_NAME=appdb
DB_USER=admin
DB_PASSWORD=supersecretpassword
REDIS_PASSWORD=redissecret
GRAFANA_PASSWORD=grafanaadmin
EOF

# Start production stack
docker compose up -d

# Start monitoring
docker compose -f docker-compose.monitoring.yml up -d

# Check all services
docker compose ps
docker compose -f docker-compose.monitoring.yml ps

# Load test to see metrics!
for i in {1..100}; do
  curl -s http://localhost/api/tasks > /dev/null &
done
wait

# View in Grafana
open http://localhost:3001
# Login: admin / grafanaadmin
# Add Prometheus datasource: http://prometheus:9090
# Import dashboard ID: 193 (Docker monitoring)
```

### What You Learned

- ✅ Nginx reverse proxy with load balancing
- ✅ SSL/TLS termination in Nginx
- ✅ API clustering (3 instances)
- ✅ Rate limiting in Nginx
- ✅ Internal Docker networks (security)
- ✅ cAdvisor + Prometheus + Grafana monitoring
- ✅ YAML anchors for DRY compose files
- ✅ Rolling deployment script
- ✅ Health checks at every layer
- ✅ Resource limits on every service

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| Containerize API | 🟢 Beginner | Books API in Docker | Dockerfile, build, run, push |
| Multi-Container App | 🟡 Intermediate | Tasks API + PostgreSQL + Redis | Compose, volumes, networks |
| Production Stack | 🔴 Advanced | Full prod with Nginx + Monitoring | SSL, clustering, Prometheus |

> 💪 **Challenge**: Deploy Project 3 on an AWS EC2 free tier instance. Add your domain, get a real SSL cert from Let's Encrypt, and watch the Grafana dashboard show real traffic. That's a portfolio project that stands out!