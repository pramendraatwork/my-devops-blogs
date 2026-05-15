---
title: "Docker — Core Concepts and Daily Commands"
date: 2024-01-20
draft: false
description: "Docker fundamentals: images, containers, volumes, networking, and Dockerfile best practices."
categories: ["docker"]
tags: ["docker", "containers", "dockerfile", "compose"]
showToc: true
---

## Core concepts

| Concept | What it is |
|---|---|
| **Image** | Read-only template (like a class) |
| **Container** | Running instance of an image |
| **Dockerfile** | Instructions to build an image |
| **Volume** | Persistent storage for containers |

## Essential commands

```bash
docker pull nginx:latest
docker build -t myapp:v1 .
docker run -d -p 8080:80 --name web nginx
docker ps && docker ps -a
docker logs -f web
docker exec -it web bash
docker stop web && docker rm web
docker system prune -a
```

## Dockerfile best practices

```dockerfile
FROM node:20.11-alpine3.19
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 3000
CMD ["node", "server.js"]
```

## Multi-stage build

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

## Docker Compose

```yaml
version: "3.9"
services:
  app:
    build: .
    ports: ["3000:3000"]
    depends_on: [postgres]
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

## Things that got me

- Layer caching: rarely-changing stuff goes at top of Dockerfile
- Always create `.dockerignore` — exclude node_modules, .git
- Stopped containers are NOT deleted — use `--rm` flag
