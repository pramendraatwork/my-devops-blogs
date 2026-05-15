---
title: "Docker Part 2 — Advanced Guide to Production Docker"
date: 2024-03-05
draft: false
description: "Advanced Docker: multi-stage builds, security, networking deep dive, monitoring, production patterns, and a complete full-stack project with CI/CD."
categories: ["docker"]
tags: ["docker", "advanced", "security", "networking", "production", "devops"]
showToc: true
TocOpen: true
---

## 1. Multi-Stage Builds — Smaller, Faster, Safer 🏗️

Multi-stage builds let you use multiple `FROM` statements. Each stage can copy files from the previous one. Result: **tiny production images** with no build tools.

### Why It Matters

```
WITHOUT MULTI-STAGE:
────────────────────────────────────────────────
Development image contains:
  • Node.js runtime          ← needed in prod
  • npm / node_modules       ← build tool (not needed in prod!)
  • TypeScript compiler      ← build tool (not needed in prod!)
  • Source maps              ← dev tool (not needed in prod!)
  • Test files               ← never needed in prod!

Image size: ~1.2 GB 😱

WITH MULTI-STAGE:
────────────────────────────────────────────────
Stage 1 (builder):  Full build environment
                    Build the app → generates /dist

Stage 2 (runner):   Minimal runtime only
                    Copy /dist from builder
                    No build tools at all!

Image size: ~85 MB ✅ (93% smaller!)
```

### Node.js Multi-Stage Build

```dockerfile
# ── Stage 1: Install & Build ──────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /build

# Install ALL deps (including devDependencies for building)
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build        # creates /build/dist

# ── Stage 2: Production Runtime ──────────────────────────────
FROM node:20-alpine AS runner

WORKDIR /app

# Only install PRODUCTION dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy ONLY the built output from builder stage
COPY --from=builder /build/dist ./dist

# Security: non-root user
RUN addgroup -S app && adduser -S app -G app
USER app

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/app.js"]
```

### Go Multi-Stage Build (Extreme Optimization)

```dockerfile
# Stage 1: Build the Go binary
FROM golang:1.22-alpine AS builder

WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app ./cmd/server

# Stage 2: Scratch — literally empty image!
FROM scratch

COPY --from=builder /build/app /app
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

EXPOSE 8080
ENTRYPOINT ["/app"]

# Final image size: ~8 MB!! 🤯
```

### Python Multi-Stage Build

```dockerfile
# Stage 1: Build dependencies
FROM python:3.12-slim AS builder

WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim AS runner

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local

# Copy application
COPY src/ ./src/

ENV PATH=/root/.local/bin:$PATH
ENV PYTHONUNBUFFERED=1

RUN useradd -r -s /bin/false appuser
USER appuser

EXPOSE 8000
CMD ["python", "-m", "uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Build Arguments — Flexible Builds

```dockerfile
FROM node:20-alpine

ARG NODE_ENV=production
ARG BUILD_VERSION=1.0.0
ARG BUILD_DATE

ENV NODE_ENV=$NODE_ENV

LABEL version=$BUILD_VERSION
LABEL build-date=$BUILD_DATE

RUN echo "Building version $BUILD_VERSION for $NODE_ENV"

COPY . .
RUN npm ci
CMD ["node", "app.js"]
```

```bash
# Build with custom args
docker build \
  --build-arg NODE_ENV=staging \
  --build-arg BUILD_VERSION=2.1.0 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t myapp:2.1.0 .
```

---

## 2. Docker Networking Deep Dive 🌐

### Network Types

```
DOCKER NETWORK TYPES
──────────────────────────────────────────────────────────────

BRIDGE (default)
────────────────
Host Machine
│
├── Docker bridge (docker0) — 172.17.0.1
│   ├── Container 1 — 172.17.0.2
│   ├── Container 2 — 172.17.0.3
│   └── Container 3 — 172.17.0.4
│
Containers can talk to each other via IP.
Reach host via 172.17.0.1

HOST
────────────────
Container shares host's network directly.
No isolation. Container port = host port.
Fastest performance. Use for high-performance needs.
docker run --network host nginx

NONE
────────────────
Completely isolated. No network access.
docker run --network none myapp

OVERLAY
────────────────
Spans multiple Docker hosts (Swarm/K8s).
Containers on different machines talk to each other.
Used in production clusters.

MACVLAN
────────────────
Container gets its own MAC address and IP.
Appears as physical device on the network.
Used for legacy apps that need direct network access.
```

### Custom Bridge Networks — The Right Way

```bash
# Create custom network
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  myapp-network

# Run containers on custom network
docker run -d \
  --name postgres \
  --network myapp-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:16

docker run -d \
  --name api \
  --network myapp-network \
  -p 3000:3000 \
  -e DB_HOST=postgres \     # ← use container NAME as hostname!
  myapp:latest

# Why custom networks are better than default bridge:
# ✅ Containers find each other by NAME (not IP)
# ✅ Better isolation
# ✅ DNS resolution built-in
# ✅ Can connect/disconnect without restart
```

### Network Troubleshooting

```bash
# Inspect network
docker network inspect myapp-network

# See which networks a container is on
docker inspect api --format='{{json .NetworkSettings.Networks}}'

# Test connectivity between containers
docker exec api ping postgres           # should work if same network
docker exec api curl http://postgres:5432

# Add container to additional network
docker network connect another-network api

# Remove from network
docker network disconnect myapp-network api

# DNS lookup inside container
docker exec api nslookup postgres
docker exec api cat /etc/hosts
docker exec api cat /etc/resolv.conf
```

---

## 3. Docker Security 🔒

Security is often ignored by beginners but critical in production.

### Security Checklist

```
DOCKER SECURITY CHECKLIST
──────────────────────────────────────────────────────────────
✅ Never run as root
✅ Use official base images
✅ Pin image versions (not :latest)
✅ Scan images for vulnerabilities
✅ Use .dockerignore
✅ Don't store secrets in images
✅ Use read-only filesystems where possible
✅ Set resource limits
✅ Use COPY not ADD (unless you need archive extraction)
✅ Keep images small (smaller = smaller attack surface)
✅ Enable Docker Content Trust
✅ Use non-root user
```

### Running as Non-Root User

```dockerfile
# ❌ BAD — running as root
FROM node:20-alpine
WORKDIR /app
COPY . .
RUN npm ci
CMD ["node", "app.js"]
# If attacker breaks out of container → they're root on host!

# ✅ GOOD — dedicated non-root user
FROM node:20-alpine

WORKDIR /app

# Create user before copying files
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 -G nodejs

COPY package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

EXPOSE 3000
CMD ["node", "app.js"]
```

### Secrets Management — Never in Dockerfile!

```dockerfile
# ❌ NEVER DO THIS — secret baked into image forever!
ENV API_KEY=super-secret-key-12345
RUN curl -H "Authorization: $API_KEY" https://api.example.com

# ✅ CORRECT — inject at runtime
CMD ["node", "app.js"]
```

```bash
# Pass secrets at runtime
docker run -d \
  -e API_KEY=$API_KEY \
  -e DB_PASSWORD=$DB_PASSWORD \
  myapp

# Or use Docker secrets (Swarm)
echo "my-secret-password" | docker secret create db_password -
docker service create \
  --secret db_password \
  myapp
# Secret available at /run/secrets/db_password inside container

# Or use .env file (don't commit this!)
docker run --env-file .env myapp
```

### Image Scanning

```bash
# Scan with Docker Scout (built into Docker)
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Scan with Trivy (popular open source)
# Install Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Scan image
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest
trivy image --format json myapp:latest > report.json

# Scan in CI/CD (GitHub Actions)
# uses: aquasecurity/trivy-action@master
# with:
#   image-ref: myapp:latest
#   severity: CRITICAL
#   exit-code: 1    # fail pipeline if CRITICAL found
```

### Read-Only Containers

```bash
# Run with read-only filesystem
docker run -d \
  --read-only \
  --tmpfs /tmp \              # allow writes to /tmp only
  --tmpfs /var/run \
  -p 3000:3000 \
  myapp

# Set resource limits
docker run -d \
  --memory="512m" \           # max 512MB RAM
  --memory-swap="512m" \      # disable swap
  --cpus="0.5" \              # max 50% of one CPU
  --pids-limit=100 \          # max 100 processes
  myapp
```

---

## 4. Docker Volumes — Data Persistence 💾

```
VOLUME TYPES:
──────────────────────────────────────────────────────────────

NAMED VOLUMES (Docker managed)
docker volume create mydata
docker run -v mydata:/app/data myapp

  Host: /var/lib/docker/volumes/mydata/_data
  Best for: Databases, persistent app data
  ✅ Portable, easy to backup, Docker manages location

BIND MOUNTS (Host path)
docker run -v $(pwd)/data:/app/data myapp

  Maps exact host path to container
  Best for: Development (live reload), config files
  ✅ See changes immediately, easy to access from host

TMPFS MOUNTS (In-memory)
docker run --tmpfs /app/temp myapp

  Stored in host memory only
  Data gone when container stops
  Best for: Sensitive data, temp files, performance
  ✅ Fast, never written to disk
```

### Volume Best Practices

```bash
# Backup a volume
docker run --rm \
  -v pgdata:/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /source .

# Restore a volume
docker run --rm \
  -v pgdata:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /target

# Copy volume to new volume
docker run --rm \
  -v old-volume:/from \
  -v new-volume:/to \
  alpine cp -r /from/. /to

# Inspect volume
docker volume inspect pgdata
```

---

## 5. Docker Resource Management 📊

### Setting Limits

```bash
# Memory limits
docker run -d \
  --memory="256m" \           # hard limit
  --memory-reservation="128m" \  # soft limit (warning)
  --oom-kill-disable \        # don't kill if OOM (use carefully)
  nginx

# CPU limits
docker run -d \
  --cpus="1.5" \              # 1.5 CPU cores max
  --cpu-shares=512 \          # relative weight (default 1024)
  --cpuset-cpus="0,1" \       # pin to specific CPUs
  nginx

# Combined
docker run -d \
  --name api \
  --memory="512m" \
  --cpus="0.5" \
  --restart unless-stopped \
  -p 3000:3000 \
  myapp:latest
```

### In Docker Compose

```yaml
version: "3.9"
services:
  api:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    restart: unless-stopped
```

---

## 6. Docker Monitoring 📈

### Built-in Stats

```bash
# Live stats for all containers
docker stats

# Stats for specific container
docker stats api postgres redis

# One-time snapshot (non-streaming)
docker stats --no-stream

# Custom format
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```

### Health Checks

```dockerfile
# In Dockerfile
HEALTHCHECK --interval=30s \
            --timeout=10s \
            --start-period=40s \
            --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

```yaml
# In docker-compose.yml
services:
  api:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

```bash
# Check health status
docker ps                           # shows health in STATUS column
docker inspect api | grep -A 10 Health
```

### Prometheus + cAdvisor (Production Monitoring)

```yaml
# docker-compose.monitoring.yml
version: "3.9"
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafanadata:/var/lib/grafana

volumes:
  grafanadata:
```

---

## 7. Production Full-Stack Project 🏭

A complete production-ready setup: React frontend + Node.js API + PostgreSQL + Redis + Nginx.

### Project Structure

```
fullstack-app/
├── frontend/
│   ├── src/
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   ├── src/
│   └── Dockerfile
├── nginx/
│   └── nginx.conf            ← reverse proxy
├── docker-compose.yml        ← development
├── docker-compose.prod.yml   ← production
└── .github/
    └── workflows/
        └── deploy.yml        ← CI/CD
```

### Production Docker Compose

```yaml
# docker-compose.prod.yml
version: "3.9"

services:
  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/var/www/static
    depends_on:
      - frontend
      - backend
    restart: always

  # React frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: runner
    container_name: frontend
    expose:
      - "80"
    restart: always

  # Node.js backend
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: runner
    container_name: backend
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://admin:${DB_PASSWORD}@postgres:5432/appdb
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: always

  # PostgreSQL
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./backend/migrations:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

  # Redis cache
  redis:
    image: redis:7-alpine
    container_name: redis
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redisdata:/data
    restart: always

volumes:
  pgdata:
  redisdata:
  static_files:

networks:
  default:
    name: appnetwork
```

### Nginx Reverse Proxy Config

```nginx
# nginx/nginx.conf
upstream frontend {
    server frontend:80;
}

upstream backend {
    server backend:3000;
}

server {
    listen 80;
    server_name yourdomain.com;

    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Backend API
    location /api/ {
        proxy_pass http://backend/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # WebSocket support
    location /ws/ {
        proxy_pass http://backend/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### CI/CD Pipeline for Docker

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_PREFIX: ghcr.io/${{ github.repository_owner }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: ${{ env.IMAGE_PREFIX }}/backend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build and push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: ${{ env.IMAGE_PREFIX }}/frontend:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ubuntu
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /app
            export IMAGE_TAG=${{ github.sha }}
            export DB_PASSWORD=${{ secrets.DB_PASSWORD }}
            export REDIS_PASSWORD=${{ secrets.REDIS_PASSWORD }}
            export JWT_SECRET=${{ secrets.JWT_SECRET }}

            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d --remove-orphans
            docker system prune -f
```

---

## 8. Docker Troubleshooting 🔧

```bash
# Container won't start
docker logs container-name             # check error logs
docker inspect container-name          # check config
docker events                          # real-time events

# Permission denied
docker exec -it --user root container bash   # enter as root
ls -la /app                                  # check permissions
chown -R appuser:appuser /app               # fix ownership

# Container exits immediately
docker run -it myapp sh                # override CMD to debug
docker run --entrypoint sh myapp       # override entrypoint too

# Network issues
docker exec container ping other-container    # test connectivity
docker exec container nslookup other-container # DNS check
docker network inspect network-name           # check network

# Image too large
docker history myapp:latest           # see layer sizes
dive myapp:latest                     # interactive layer explorer
# Install dive: https://github.com/wagoodman/dive

# Out of disk space
docker system df                      # see disk usage
docker system prune -a --volumes      # nuclear cleanup

# Build cache issues
docker build --no-cache -t myapp .    # force rebuild all layers

# Common error messages:
# "No space left on device"    → docker system prune -a
# "Port already in use"        → lsof -i :PORT, kill process
# "Permission denied"          → check user, volume permissions
# "Cannot connect to daemon"   → sudo systemctl start docker
# "Image not found"            → docker pull image, check typo
```

---

## 9. Docker Advanced Cheatsheet 📋

```bash
# MULTI-STAGE
docker build --target builder -t myapp:builder .  # build specific stage
docker build --target runner -t myapp:prod .       # build final stage

# SECURITY
docker scan myapp:latest                           # scan vulnerabilities
trivy image myapp:latest                           # trivy scan
docker run --read-only myapp                       # read-only fs
docker run --user 1001:1001 myapp                  # specific UID/GID
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp  # drop capabilities

# NETWORKING
docker network create --driver overlay mynet       # overlay network
docker network connect mynet container             # connect container
docker run --add-host host.docker.internal:host-gateway myapp  # reach host

# RESOURCES
docker run --memory="512m" --cpus="0.5" myapp
docker update --memory="1g" running-container      # update live!

# REGISTRY
docker tag myapp:latest registry.example.com/myapp:latest
docker push registry.example.com/myapp:latest
docker pull registry.example.com/myapp:latest
docker login registry.example.com

# DEBUGGING
docker run --rm -it --entrypoint sh myapp          # debug image
docker diff container-name                         # see filesystem changes
docker commit container-name debug-image           # save container as image
docker export container > backup.tar               # export filesystem
docker import backup.tar myapp:restored            # import filesystem

# COMPOSE ADVANCED
docker compose -f docker-compose.yml \
               -f docker-compose.prod.yml up -d    # merge compose files
docker compose --profile monitoring up -d          # use profiles
docker compose config                              # validate & view merged config
docker compose top                                 # processes in each service
```

---

## What's Next? 🚀

You've mastered Docker! The natural next step:

- **Kubernetes** — orchestrate hundreds of containers across clusters
- **Docker Swarm** — simpler orchestration built into Docker
- **AWS ECS/EKS** — managed container services on AWS
- **Helm** — package manager for Kubernetes

> 💪 **Final challenge**: Take the production full-stack project above, deploy it to a real server (AWS EC2 free tier!), set up the CI/CD pipeline, and you'll have a real DevOps project to show in interviews!