---
title: "Docker Part 1 — Beginner's Complete Guide to Containerization"
date: 2024-03-01
draft: false
description: "Docker from absolute zero: what containers are, how Docker works, all essential commands, and your first containerized project. Simple, visual, easy to understand."
categories: ["docker"]
tags: ["docker", "containers", "beginner", "dockerfile", "devops"]
showToc: true
TocOpen: true
---

## 1. What is Docker? 🐳

Before Docker, deploying an application was a nightmare. Every developer had a different machine, different OS, different versions of software. The famous excuse was:

> 😤 **"But it works on MY machine!"**

Docker killed that excuse forever.

### The Problem Docker Solves

```
WITHOUT DOCKER — The "Works on My Machine" Problem
────────────────────────────────────────────────────────────

Developer's Laptop        Staging Server         Production Server
─────────────────         ──────────────         ─────────────────
Ubuntu 22.04              CentOS 7               Amazon Linux 2
Node.js 20                Node.js 14  ← different!  Node.js 18 ← different!
npm 10                    npm 6       ← different!  npm 8  ← different!
MongoDB 7                 MongoDB 4   ← different!  MongoDB 5 ← different!

Result: App works locally, BREAKS in production! 🔥

────────────────────────────────────────────────────────────

WITH DOCKER — Ship the entire environment!
────────────────────────────────────────────────────────────

Developer's Laptop        Staging Server         Production Server
─────────────────         ──────────────         ─────────────────
┌─────────────────┐       ┌────────────────┐     ┌────────────────┐
│   CONTAINER     │       │   CONTAINER    │     │   CONTAINER    │
│  Node.js 20     │  ═══  │  Node.js 20   │ ═══ │  Node.js 20   │
│  npm 10         │       │  npm 10       │     │  npm 10       │
│  MongoDB 7      │       │  MongoDB 7    │     │  MongoDB 7    │
│  Your App       │       │  Your App     │     │  Your App     │
└─────────────────┘       └────────────────┘     └────────────────┘
  Identical!                Identical!              Identical!

Result: Works everywhere, every time! ✅
```

---

## 2. Virtual Machines vs Docker Containers 🆚

This is the most important concept to understand first.

```
VIRTUAL MACHINES                    DOCKER CONTAINERS
────────────────────────────────────────────────────────────

┌──────────────────────┐           ┌──────────────────────┐
│      App A           │           │  App A  │  App B      │
├──────────────────────┤           ├─────────┴────────────-┤
│   Guest OS (Ubuntu)  │           │    Docker Engine       │
├──────────────────────┤           ├──────────────────────-─┤
│     Hypervisor       │           │      Host OS           │
├──────────────────────┤           ├──────────────────────-─┤
│   Physical Hardware  │           │   Physical Hardware     │
└──────────────────────┘           └──────────────────────-─┘

Size: GBs                          Size: MBs
Boot time: Minutes                 Boot time: Seconds
Isolated OS: Yes (heavy)           Shared OS kernel (light)
Resource usage: High               Resource usage: Low
```

| Feature | Virtual Machine | Docker Container |
|---|---|---|
| **Size** | GBs | MBs |
| **Startup** | Minutes | Seconds |
| **OS** | Full guest OS | Shared host kernel |
| **Isolation** | Complete | Process-level |
| **Performance** | Lower | Near-native |
| **Use case** | Full OS isolation | App isolation |

> 💡 **Simple analogy**:
> - VM = Buying a whole new house for each family
> - Container = Apartments in the same building (shared infrastructure, separate spaces)

---

## 3. Docker Core Concepts 🧠

```
┌─────────────────────────────────────────────────────────────┐
│                    DOCKER ECOSYSTEM                         │
│                                                             │
│   Dockerfile  ──build──→  Image  ──run──→  Container       │
│   (recipe)               (template)        (running app)    │
│                               │                             │
│                          push/pull                          │
│                               │                             │
│                          Registry                           │
│                      (Docker Hub / ECR)                     │
└─────────────────────────────────────────────────────────────┘
```

### The 5 Key Concepts

**1. Dockerfile** — The Recipe 📝
```
Instructions to build your image.
Like a recipe — tells Docker exactly what to install and how to set up.
```

**2. Image** — The Template 📦
```
A read-only snapshot built from Dockerfile.
Like a class in programming — you define it once, use it many times.
Think of it as a frozen state of your app + all dependencies.
```

**3. Container** — The Running Instance 🏃
```
A running instance of an image.
Like an object created from a class.
You can run 100 containers from one image — all identical!
```

**4. Registry** — The Warehouse 🏪
```
Stores and shares Docker images.
Docker Hub = public registry (like GitHub for images)
AWS ECR, GCR, ACR = private registries for companies
```

**5. Volume** — Persistent Storage 💾
```
Containers are stateless — when they die, data dies too.
Volumes store data outside the container — it survives restarts.
Used for databases, logs, uploads.
```

---

## 4. Installing Docker 🔧

### Ubuntu/Debian (WSL2 / Linux Server)

```bash
# Step 1: Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Step 2: Install prerequisites
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Step 3: Add Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Step 4: Add Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Step 5: Install Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Step 6: Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Step 7: Run without sudo
sudo usermod -aG docker $USER
newgrp docker    # apply group change now

# Verify
docker --version
docker run hello-world    # test installation ✅
```

### Windows (Docker Desktop)
```
1. Download Docker Desktop from docker.com
2. Install and restart
3. Enable WSL2 backend in settings
4. Done! Works with WSL2 terminal
```

---

## 5. Essential Docker Commands 💻

### Images

```bash
# Pull an image from Docker Hub
docker pull nginx                    # pull latest
docker pull nginx:1.25               # pull specific version
docker pull node:20-alpine           # Alpine = smaller image

# List local images
docker images
docker image ls
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Build image from Dockerfile
docker build -t myapp .              # build with tag
docker build -t myapp:v1 .           # with version
docker build -t myapp:v1 -f Dockerfile.prod .  # specific Dockerfile

# Remove images
docker rmi nginx                     # remove by name
docker rmi nginx:1.25                # remove specific version
docker image prune                   # remove dangling images
docker image prune -a                # remove ALL unused images

# Image info
docker inspect nginx                 # detailed JSON info
docker history nginx                 # show image layers
```

### Containers

```bash
# Run containers
docker run nginx                     # run (attached — blocks terminal)
docker run -d nginx                  # run detached (background) ✅
docker run -d -p 8080:80 nginx       # map port host:container
docker run -d --name webserver nginx # give container a name
docker run -it ubuntu bash           # interactive terminal
docker run --rm ubuntu echo "hello"  # auto-delete when done

# Port mapping explained:
# -p 8080:80
#    ↑    ↑
#  host  container
# Open localhost:8080 in browser → hits container port 80

# List containers
docker ps                            # running containers
docker ps -a                         # ALL containers (including stopped)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Container lifecycle
docker stop webserver                # graceful stop (SIGTERM)
docker start webserver               # start stopped container
docker restart webserver             # stop + start
docker kill webserver                # force kill (SIGKILL)
docker pause webserver               # pause (freeze)
docker unpause webserver             # unpause

# Remove containers
docker rm webserver                  # remove stopped container
docker rm -f webserver               # force remove running container
docker container prune               # remove ALL stopped containers

# Logs
docker logs webserver                # print logs
docker logs -f webserver             # follow logs (like tail -f)
docker logs --tail 50 webserver      # last 50 lines
docker logs --since 1h webserver     # logs from last hour

# Execute commands in running container
docker exec webserver ls /           # run one command
docker exec -it webserver bash       # interactive shell ← use this ALL the time
docker exec -it webserver sh         # if bash not available

# Copy files
docker cp webserver:/etc/nginx/nginx.conf ./nginx.conf  # from container
docker cp ./index.html webserver:/usr/share/nginx/html/ # to container

# Container info
docker inspect webserver             # full JSON details
docker stats                         # live CPU/memory usage
docker stats webserver               # specific container
docker top webserver                 # processes inside container
```

### Volumes

```bash
# Named volumes (Docker manages location)
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker volume prune                  # remove unused volumes

# Run with named volume
docker run -d \
  --name postgres \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

# Bind mounts (map host directory)
docker run -d \
  --name nginx \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -p 8080:80 \
  nginx
# Changes to ./html on host immediately appear in container!
# :ro = read-only (container can't write)
```

### Networks

```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Run containers on same network
docker run -d --name db --network mynetwork postgres:16
docker run -d --name app --network mynetwork myapp
# Now "app" can reach "db" using hostname "db" ✅

# Inspect network
docker network inspect mynetwork

# Remove network
docker network rm mynetwork
docker network prune
```

### System Commands

```bash
# System info
docker info                          # Docker system info
docker system df                     # disk usage
docker version                       # Docker version

# Cleanup (run regularly!)
docker system prune                  # remove stopped containers, dangling images
docker system prune -a               # also remove unused images
docker system prune -a --volumes     # also remove volumes (CAREFUL!)
```

---

## 6. Writing Dockerfiles 📝

A Dockerfile is a text file with instructions to build an image. Each instruction creates a **layer**.

### Dockerfile Instructions

```dockerfile
# FROM — base image (always first)
FROM node:20-alpine

# WORKDIR — set working directory
WORKDIR /app

# COPY — copy files from host to image
COPY package*.json ./
COPY src/ ./src/

# ADD — like COPY but can extract archives and fetch URLs
ADD app.tar.gz /app/

# RUN — execute command during BUILD
RUN npm ci --only=production
RUN apt-get update && apt-get install -y curl

# ENV — set environment variables
ENV NODE_ENV=production
ENV PORT=3000

# ARG — build-time variables (not in final image)
ARG VERSION=1.0.0
RUN echo "Building version $VERSION"

# EXPOSE — document which port the app uses
EXPOSE 3000

# VOLUME — declare mount points
VOLUME ["/app/data"]

# USER — switch to non-root user
USER node

# CMD — default command when container starts (can be overridden)
CMD ["node", "src/app.js"]

# ENTRYPOINT — always runs (CMD becomes arguments)
ENTRYPOINT ["node"]
CMD ["src/app.js"]

# HEALTHCHECK — how Docker checks if container is healthy
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# LABEL — metadata
LABEL maintainer="pramendraatwork@gmail.com"
LABEL version="1.0"
```

### Layer Caching — The Most Important Optimization

```dockerfile
# ❌ BAD — cache breaks every time ANY file changes
FROM node:20-alpine
WORKDIR /app
COPY . .                    ← copies everything including source code
RUN npm install             ← runs every time! even if deps didn't change

# ✅ GOOD — deps cached separately from source code
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./       ← only package files first
RUN npm ci                  ← only reruns if package.json changed!
COPY . .                    ← source code copied after
```

```
HOW LAYER CACHING WORKS:
─────────────────────────────────────────────────────
Layer 1: FROM node:20-alpine        ← cached ✅
Layer 2: WORKDIR /app               ← cached ✅
Layer 3: COPY package*.json ./      ← cached ✅ (if package.json unchanged)
Layer 4: RUN npm ci                 ← cached ✅ (skipped! uses cache)
Layer 5: COPY . .                   ← rebuilt (source changed)
Layer 6: EXPOSE 3000                ← rebuilt

Only layers AFTER a change get rebuilt. Put stable things first!
```

---

## 7. Your First Docker Project 🚀

Let's containerize a real Node.js web app from scratch!

### Project: Containerized Todo API

```
todo-api/
├── src/
│   └── app.js
├── Dockerfile
├── .dockerignore
└── package.json
```

**Step 1: Create the app**

```bash
mkdir todo-api && cd todo-api
npm init -y
npm install express
```

```javascript
// src/app.js
const express = require('express');
const app = express();
app.use(express.json());

// In-memory storage (for simplicity)
let todos = [
  { id: 1, task: 'Learn Docker', done: false },
  { id: 2, task: 'Build something cool', done: false }
];

// Routes
app.get('/', (req, res) => {
  res.json({ message: '🐳 Todo API running in Docker!', version: '1.0' });
});

app.get('/todos', (req, res) => {
  res.json(todos);
});

app.post('/todos', (req, res) => {
  const todo = { id: Date.now(), task: req.body.task, done: false };
  todos.push(todo);
  res.status(201).json(todo);
});

app.put('/todos/:id', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  if (!todo) return res.status(404).json({ error: 'Not found' });
  todo.done = !todo.done;
  res.json(todo);
});

app.delete('/todos/:id', (req, res) => {
  todos = todos.filter(t => t.id !== parseInt(req.params.id));
  res.json({ message: 'Deleted' });
});

app.get('/health', (req, res) => res.json({ status: 'healthy' }));

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`🚀 Server running on port ${PORT}`));
```

**Step 2: Create .dockerignore**

```bash
# .dockerignore (like .gitignore but for Docker)
node_modules/
npm-debug.log
.git/
.env
*.md
Dockerfile
.dockerignore
```

**Step 3: Write the Dockerfile**

```dockerfile
# Use official Node.js Alpine image (small!)
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy dependency files FIRST (for layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY src/ ./src/

# Create non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Document the port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q http://localhost:3000/health -O - || exit 1

# Start the app
CMD ["node", "src/app.js"]
```

**Step 4: Build the image**

```bash
# Build it
docker build -t todo-api:v1 .

# Check the image was created
docker images | grep todo-api

# See image layers
docker history todo-api:v1
```

**Step 5: Run it**

```bash
# Run the container
docker run -d \
  --name todo-app \
  -p 3000:3000 \
  todo-api:v1

# Check it's running
docker ps

# View logs
docker logs todo-app

# Test it!
curl http://localhost:3000/
curl http://localhost:3000/todos
curl -X POST http://localhost:3000/todos \
  -H "Content-Type: application/json" \
  -d '{"task": "Master Docker"}'
```

**Step 6: Interact with it**

```bash
# Shell into the running container
docker exec -it todo-app sh

# Inside container — explore!
ls /app
cat /app/package.json
ps aux               # see running processes
whoami               # should show appuser (not root!)
exit

# Stop and remove
docker stop todo-app
docker rm todo-app
```

---

## 8. Docker Compose — Multi-Container Apps 🎼

Most real apps need multiple services: API + Database + Cache. Docker Compose runs them all together.

```yaml
# docker-compose.yml
version: "3.9"

services:
  # Our API
  api:
    build: .                        # build from Dockerfile in current dir
    container_name: todo-api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGO_URL=mongodb://mongo:27017/todos
      - REDIS_URL=redis://redis:6379
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  # MongoDB database
  mongo:
    image: mongo:7
    container_name: todo-mongo
    volumes:
      - mongodata:/data/db           # persist data!
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost/test
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: todo-redis
    volumes:
      - redisdata:/data

  # Mongo Express — web UI for MongoDB
  mongo-express:
    image: mongo-express
    container_name: todo-mongo-ui
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
    depends_on:
      - mongo

volumes:
  mongodata:    # named volumes persist across restarts
  redisdata:
```

```bash
# Compose commands
docker compose up -d              # start all services
docker compose up -d api          # start specific service
docker compose down               # stop and remove containers
docker compose down -v            # also remove volumes
docker compose ps                 # status of all services
docker compose logs -f            # logs from all services
docker compose logs -f api        # logs from one service
docker compose exec api sh        # shell into service
docker compose restart api        # restart one service
docker compose build              # rebuild images
docker compose pull               # pull latest images
```

---

## 9. Docker Cheatsheet 📋

```bash
# IMAGES
docker pull image:tag             # download
docker build -t name:tag .        # build
docker images                     # list
docker rmi image                  # delete
docker image prune -a             # clean unused

# CONTAINERS
docker run -d -p 8080:80 --name web nginx   # run
docker ps / docker ps -a                     # list
docker stop/start/restart name               # lifecycle
docker rm -f name                            # force delete
docker logs -f name                          # logs
docker exec -it name bash                    # shell in

# VOLUMES
docker volume create name                    # create
docker run -v name:/path image               # use volume
docker run -v $(pwd):/path image             # bind mount
docker volume ls / prune                     # list/clean

# COMPOSE
docker compose up -d                         # start
docker compose down                          # stop
docker compose logs -f                       # logs
docker compose exec service bash            # shell
docker compose ps                            # status

# CLEANUP
docker system prune -a                       # clean everything
docker container prune                       # stopped containers
docker image prune -a                        # unused images
docker volume prune                          # unused volumes
```

---

## What's in Part 2? 🚀

Part 1 covered the fundamentals. **Docker Part 2** goes deeper:

- 🏗️ **Multi-stage builds** — smaller production images
- 🔒 **Docker security** — scanning, non-root, secrets
- 🌐 **Docker networking** — bridge, host, overlay networks
- 📊 **Docker monitoring** — stats, cAdvisor, Prometheus
- ☸️ **Docker + Kubernetes** — from compose to orchestration
- 🏭 **Production patterns** — health checks, resource limits, logging
- 🚀 **Real production project** — full stack app with CI/CD

> 💪 **Practice now**: Containerize any app you've built before. Take a simple Node.js or Python project, write a Dockerfile, build it, run it. That hands-on experience is worth more than reading 10 articles!