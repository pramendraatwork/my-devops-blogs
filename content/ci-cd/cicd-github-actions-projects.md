---
title: "CI/CD Projects with GitHub Actions — Beginner to Advanced (3 Real Projects)"
date: 2024-03-23
draft: false
description: "3 hands-on CI/CD projects using GitHub Actions: beginner Node.js pipeline, intermediate full-stack with Docker, and advanced multi-environment Kubernetes deployment."
categories: ["ci-cd"]
tags: ["github-actions", "ci-cd", "pipeline", "projects", "devops", "automation", "docker", "kubernetes"]
showToc: true
TocOpen: true
---

## Why GitHub Actions? 🎯

GitHub Actions is the fastest way to get CI/CD running — zero server setup, free for public repos, and deeply integrated with GitHub. These 3 projects go from your first workflow to a full production-grade Kubernetes deployment pipeline.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → Node.js App CI Pipeline
Project 2 (Intermediate) → Full Stack App with Docker & Deploy
Project 3 (Advanced)     → Multi-Environment Kubernetes Pipeline
──────────────────────────────────────────────────────────────
```

---

## Project 1: Node.js App CI Pipeline 🟢 Beginner

### What You'll Build

A complete CI pipeline for a Node.js REST API — runs on every push and PR, installs dependencies, lints code, runs tests, checks coverage, and shows results right in GitHub.

```
PIPELINE FLOW:
──────────────────────────────────────────────────────────────
Developer pushes code / opens PR
              │
              ▼
GitHub Actions triggered automatically
              │
              ▼
┌─────────────────────────────────────┐
│            CI JOB                   │
│                                     │
│  1. Checkout code                   │
│  2. Setup Node.js (v18, v20, v22)   │  ← matrix build!
│  3. Install dependencies            │
│  4. Run ESLint                      │
│  5. Run Jest tests + coverage       │
│  6. Upload coverage report          │
│  7. Comment results on PR           │
└─────────────────────────────────────┘
              │
        ✅ Pass → PR can be merged
        ❌ Fail → PR blocked, dev notified
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- GitHub Actions basics (on, jobs, steps)
- Matrix builds (test multiple Node versions)
- Caching dependencies
- Uploading artifacts
- PR status checks
- Secrets management

### Project Structure

```
node-ci-pipeline/
├── .github/
│   └── workflows/
│       └── ci.yml          ← our pipeline
├── src/
│   ├── app.js
│   └── routes/
│       └── todos.js
├── tests/
│   └── todos.test.js
├── .eslintrc.json
├── jest.config.js
├── Dockerfile
└── package.json
```

### The Application

```javascript
// src/app.js
const express = require('express');
const app = express();
app.use(express.json());
app.use('/api/todos', require('./routes/todos'));

app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    version: process.env.npm_package_version || '1.0.0',
    node: process.version,
    timestamp: new Date().toISOString()
  });
});

app.use((req, res) => res.status(404).json({ error: 'Not found' }));

module.exports = app;
if (require.main === module) {
  app.listen(3000, () => console.log('🚀 Running on port 3000'));
}
```

```javascript
// src/routes/todos.js
const router = require('express').Router();

let todos = [];
let nextId = 1;

router.get('/', (req, res) => {
  const { done } = req.query;
  const result = done !== undefined
    ? todos.filter(t => t.done === (done === 'true'))
    : todos;
  res.json(result);
});

router.post('/', (req, res) => {
  const { task } = req.body;
  if (!task || task.trim() === '') {
    return res.status(400).json({ error: 'Task is required' });
  }
  const todo = { id: nextId++, task: task.trim(), done: false,
                 createdAt: new Date().toISOString() };
  todos.push(todo);
  res.status(201).json(todo);
});

router.put('/:id', (req, res) => {
  const todo = todos.find(t => t.id === parseInt(req.params.id));
  if (!todo) return res.status(404).json({ error: 'Todo not found' });
  todo.done = !todo.done;
  res.json(todo);
});

router.delete('/:id', (req, res) => {
  const index = todos.findIndex(t => t.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ error: 'Todo not found' });
  todos.splice(index, 1);
  res.json({ message: 'Deleted successfully' });
});

module.exports = router;
```

```javascript
// tests/todos.test.js
const request = require('supertest');
const app = require('../src/app');

// Reset todos before each test
beforeEach(() => {
  // Reset module to clear todos array
  jest.resetModules();
});

describe('Health Check', () => {
  test('GET /health returns healthy status', async () => {
    const res = await request(app).get('/health');
    expect(res.status).toBe(200);
    expect(res.body.status).toBe('healthy');
    expect(res.body.node).toBeDefined();
  });
});

describe('Todos API', () => {
  test('GET /api/todos returns empty array initially', async () => {
    const res = await request(app).get('/api/todos');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
  });

  test('POST /api/todos creates a todo', async () => {
    const res = await request(app)
      .post('/api/todos')
      .send({ task: 'Learn GitHub Actions' });
    expect(res.status).toBe(201);
    expect(res.body.task).toBe('Learn GitHub Actions');
    expect(res.body.done).toBe(false);
    expect(res.body.id).toBeDefined();
  });

  test('POST /api/todos requires task', async () => {
    const res = await request(app)
      .post('/api/todos')
      .send({});
    expect(res.status).toBe(400);
    expect(res.body.error).toBeDefined();
  });

  test('POST /api/todos rejects empty task', async () => {
    const res = await request(app)
      .post('/api/todos')
      .send({ task: '   ' });
    expect(res.status).toBe(400);
  });

  test('PUT /api/todos/:id toggles done status', async () => {
    // First create a todo
    const create = await request(app)
      .post('/api/todos')
      .send({ task: 'Test toggle' });
    const id = create.body.id;

    // Toggle it
    const toggle = await request(app).put(`/api/todos/${id}`);
    expect(toggle.status).toBe(200);
    expect(toggle.body.done).toBe(true);
  });

  test('PUT /api/todos/:id returns 404 for unknown', async () => {
    const res = await request(app).put('/api/todos/9999');
    expect(res.status).toBe(404);
  });

  test('DELETE /api/todos/:id deletes todo', async () => {
    const create = await request(app)
      .post('/api/todos')
      .send({ task: 'To be deleted' });
    const id = create.body.id;

    const del = await request(app).delete(`/api/todos/${id}`);
    expect(del.status).toBe(200);
    expect(del.body.message).toBeDefined();
  });

  test('GET /api/unknown returns 404', async () => {
    const res = await request(app).get('/api/unknown');
    expect(res.status).toBe(404);
  });
});
```

```json
// .eslintrc.json
{
  "env": { "node": true, "es2021": true, "jest": true },
  "extends": "eslint:recommended",
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off",
    "semi": ["error", "always"],
    "quotes": ["error", "single"]
  }
}
```

```json
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  collectCoverageFrom: ['src/**/*.js'],
  coverageThreshold: {
    global: {
      lines: 70,
      functions: 70,
      branches: 60,
      statements: 70
    }
  },
  coverageReporters: ['text', 'lcov', 'html'],
  testResultsProcessor: 'jest-junit'
};
```

```json
// package.json
{
  "name": "node-ci-pipeline",
  "version": "1.0.0",
  "scripts": {
    "start": "node src/app.js",
    "test": "jest",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/ tests/",
    "lint:fix": "eslint src/ tests/ --fix"
  },
  "dependencies": {
    "express": "^4.18.0"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "jest": "^29.0.0",
    "jest-junit": "^16.0.0",
    "supertest": "^6.0.0"
  }
}
```

### The GitHub Actions CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

# Cancel previous runs on same branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # ── Job 1: Lint ────────────────────────────────────────────
  lint:
    name: 🔍 Lint Code
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

  # ── Job 2: Test Matrix ─────────────────────────────────────
  test:
    name: 🧪 Test (Node ${{ matrix.node-version }})
    runs-on: ${{ matrix.os }}
    needs: lint   # only run if lint passes

    strategy:
      fail-fast: false    # don't cancel other matrix jobs if one fails
      matrix:
        node-version: ['18', '20', '22']
        os: [ubuntu-latest]
        include:
          # Also test on Windows with Node 20
          - node-version: '20'
            os: windows-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()    # upload even if tests fail
        with:
          name: test-results-node${{ matrix.node-version }}-${{ matrix.os }}
          path: |
            junit.xml
            coverage/
          retention-days: 7

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        if: matrix.node-version == '20' && matrix.os == 'ubuntu-latest'
        with:
          file: ./coverage/lcov.info
          flags: unittests
          name: coverage-report

  # ── Job 3: Security Audit ──────────────────────────────────
  security:
    name: 🔒 Security Audit
    runs-on: ubuntu-latest
    needs: lint

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Check for known vulnerabilities
        uses: snyk/actions/node@master
        continue-on-error: true   # don't fail build, just report
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # ── Job 4: PR Comment ──────────────────────────────────────
  comment:
    name: 💬 Comment on PR
    runs-on: ubuntu-latest
    needs: [lint, test, security]
    if: github.event_name == 'pull_request'

    steps:
      - uses: actions/checkout@v4

      - name: Download test results
        uses: actions/download-artifact@v4
        with:
          name: test-results-node20-ubuntu-latest
          path: test-results/

      - name: Comment PR with results
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const comment = `
            ## ✅ CI Pipeline Results

            | Check | Status |
            |-------|--------|
            | Lint | ✅ Passed |
            | Tests (Node 18) | ✅ Passed |
            | Tests (Node 20) | ✅ Passed |
            | Tests (Node 22) | ✅ Passed |
            | Security Audit | ✅ Passed |

            **Build:** #${{ github.run_number }}
            **Commit:** ${{ github.sha }}
            `;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

### Setup & Run

```bash
# Create the project
mkdir node-ci-pipeline && cd node-ci-pipeline
git init && git checkout -b main

# Create all files above
mkdir -p src/routes tests .github/workflows

# Install dependencies
npm install

# Test locally first
npm test
npm run lint

# Push to GitHub
git add .
git commit -m "feat: add Node.js CI pipeline"
git remote add origin https://github.com/pramendraatwork/node-ci-pipeline.git
git push -u origin main

# Now open a PR and watch the pipeline run!
git checkout -b feature/add-search
# make a small change...
git commit -m "feat: add todo search"
git push origin feature/add-search
# Open PR on GitHub → watch Actions run → see comment!
```

### What You Learned

- ✅ GitHub Actions basics
- ✅ Matrix builds (multiple Node versions)
- ✅ Caching for faster builds
- ✅ Uploading artifacts
- ✅ PR status checks
- ✅ Auto-commenting on PRs
- ✅ Security scanning

---

## Project 2: Full Stack App with Docker & Deployment 🟡 Intermediate

### What You'll Build

A complete CI/CD pipeline for a full-stack app (React + Node.js API) — builds Docker images, pushes to Docker Hub, deploys to a server, and runs smoke tests after deployment.

```
PIPELINE FLOW:
──────────────────────────────────────────────────────────────
Push to main branch
        │
        ▼
┌───────────────────────────────────┐
│    Job 1: test-backend            │
│    Job 2: test-frontend           │  ← run in parallel!
└───────────────────────────────────┘
        │ (both must pass)
        ▼
┌───────────────────────────────────┐
│    Job 3: build-and-push          │
│    • Build backend Docker image   │
│    • Build frontend Docker image  │
│    • Push both to Docker Hub      │
└───────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────┐
│    Job 4: deploy-staging          │
│    • SSH to staging server        │
│    • Pull new images              │
│    • docker compose up            │
│    • Run smoke tests              │
└───────────────────────────────────┘
        │
        ▼
┌───────────────────────────────────┐
│    Job 5: deploy-production       │
│    • Requires manual approval     │
│    • Same deploy steps            │
│    • Post-deploy verification     │
└───────────────────────────────────┘
──────────────────────────────────────────────────────────────
```

### Project Structure

```
fullstack-cicd/
├── .github/
│   └── workflows/
│       └── cicd.yml
├── backend/
│   ├── src/app.js
│   ├── tests/app.test.js
│   ├── Dockerfile
│   └── package.json
├── frontend/
│   ├── src/App.jsx
│   ├── Dockerfile
│   └── package.json
├── nginx/
│   └── nginx.conf
└── docker-compose.yml
```

### Backend Dockerfile

```dockerfile
# backend/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

FROM node:20-alpine AS runner
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY --from=builder /app/src ./src
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "src/app.js"]
```

### Frontend Dockerfile

```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost/ || exit 1
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.9'

services:
  nginx:
    image: nginx:alpine
    ports:
      - '80:80'
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

  frontend:
    image: ${DOCKERHUB_USERNAME}/fullstack-frontend:${IMAGE_TAG:-latest}
    restart: unless-stopped

  backend:
    image: ${DOCKERHUB_USERNAME}/fullstack-backend:${IMAGE_TAG:-latest}
    environment:
      - NODE_ENV=production
      - PORT=3000
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U admin -d appdb']
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:
```

### The Complete CI/CD Workflow

```yaml
# .github/workflows/cicd.yml
name: Full Stack CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
  BACKEND_IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-backend
  FRONTEND_IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/fullstack-frontend
  IMAGE_TAG: ${{ github.sha }}

jobs:
  # ── Job 1: Test Backend ────────────────────────────────────
  test-backend:
    name: 🧪 Test Backend
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install backend dependencies
        run: cd backend && npm ci

      - name: Run backend lint
        run: cd backend && npm run lint

      - name: Run backend tests
        run: cd backend && npm test -- --coverage
        env:
          DATABASE_URL: postgres://postgres:testpass@localhost:5432/testdb
          NODE_ENV: test

      - name: Upload backend coverage
        uses: actions/upload-artifact@v4
        with:
          name: backend-coverage
          path: backend/coverage/
          retention-days: 5

  # ── Job 2: Test Frontend ───────────────────────────────────
  test-frontend:
    name: 🧪 Test Frontend
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json

      - name: Install frontend dependencies
        run: cd frontend && npm ci

      - name: Run frontend lint
        run: cd frontend && npm run lint

      - name: Run frontend tests
        run: cd frontend && npm test -- --watchAll=false --coverage

      - name: Build frontend (check no build errors)
        run: cd frontend && npm run build

      - name: Upload frontend coverage
        uses: actions/upload-artifact@v4
        with:
          name: frontend-coverage
          path: frontend/coverage/
          retention-days: 5

  # ── Job 3: Build & Push Docker Images ─────────────────────
  build-and-push:
    name: 🐳 Build & Push Images
    runs-on: ubuntu-latest
    needs: [test-backend, test-frontend]
    if: github.ref == 'refs/heads/main' || github.ref == 'refs/heads/develop'

    outputs:
      image-tag: ${{ env.IMAGE_TAG }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Build backend
      - name: Build & push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: |
            ${{ env.BACKEND_IMAGE }}:${{ env.IMAGE_TAG }}
            ${{ env.BACKEND_IMAGE }}:latest
          cache-from: type=gha,scope=backend
          cache-to: type=gha,mode=max,scope=backend

      # Build frontend
      - name: Build & push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.FRONTEND_IMAGE }}:${{ env.IMAGE_TAG }}
            ${{ env.FRONTEND_IMAGE }}:latest
          cache-from: type=gha,scope=frontend
          cache-to: type=gha,mode=max,scope=frontend

      - name: Image summary
        run: |
          echo "## 🐳 Docker Images Built" >> $GITHUB_STEP_SUMMARY
          echo "| Image | Tag |" >> $GITHUB_STEP_SUMMARY
          echo "|-------|-----|" >> $GITHUB_STEP_SUMMARY
          echo "| Backend | \`${{ env.IMAGE_TAG }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| Frontend | \`${{ env.IMAGE_TAG }}\` |" >> $GITHUB_STEP_SUMMARY

  # ── Job 4: Deploy to Staging ───────────────────────────────
  deploy-staging:
    name: 🚀 Deploy Staging
    runs-on: ubuntu-latest
    needs: build-and-push
    if: github.ref == 'refs/heads/main'
    environment:
      name: staging
      url: http://${{ secrets.STAGING_HOST }}

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to staging server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: deploy
          key: ${{ secrets.STAGING_SSH_KEY }}
          envs: DOCKERHUB_USERNAME,IMAGE_TAG
          script: |
            echo "🚀 Deploying to staging: $IMAGE_TAG"
            cd /app

            # Update .env
            echo "DOCKERHUB_USERNAME=$DOCKERHUB_USERNAME" > .env
            echo "IMAGE_TAG=$IMAGE_TAG" >> .env
            echo "DB_PASSWORD=${{ secrets.DB_PASSWORD }}" >> .env

            # Pull and deploy
            docker compose pull
            docker compose up -d --remove-orphans

            # Wait for services
            sleep 15

            echo "✅ Staging deployment complete!"
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          IMAGE_TAG: ${{ env.IMAGE_TAG }}

      - name: Run smoke tests on staging
        run: |
          echo "Running smoke tests on staging..."

          # Health check - backend
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            http://${{ secrets.STAGING_HOST }}/api/health)

          if [ "$STATUS" != "200" ]; then
            echo "❌ Backend health check failed: $STATUS"
            exit 1
          fi
          echo "✅ Backend healthy"

          # Frontend check
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            http://${{ secrets.STAGING_HOST }}/)

          if [ "$STATUS" != "200" ]; then
            echo "❌ Frontend check failed: $STATUS"
            exit 1
          fi
          echo "✅ Frontend healthy"

          echo "✅ All smoke tests passed!"

  # ── Job 5: Deploy to Production ────────────────────────────
  deploy-production:
    name: 🎯 Deploy Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment:
      name: production          # requires manual approval in GitHub!
      url: https://yourdomain.com

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to production
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: deploy
          key: ${{ secrets.PROD_SSH_KEY }}
          envs: DOCKERHUB_USERNAME,IMAGE_TAG
          script: |
            echo "🎯 Deploying to production: $IMAGE_TAG"
            cd /app

            # Backup current compose state
            cp docker-compose.yml docker-compose.yml.bak

            # Update .env
            echo "DOCKERHUB_USERNAME=$DOCKERHUB_USERNAME" > .env
            echo "IMAGE_TAG=$IMAGE_TAG" >> .env
            echo "DB_PASSWORD=${{ secrets.DB_PASSWORD }}" >> .env

            # Pull new images
            docker compose pull

            # Rolling deploy (no downtime)
            docker compose up -d --no-deps --remove-orphans backend
            sleep 10
            docker compose up -d --no-deps --remove-orphans frontend
            sleep 5
            docker compose up -d nginx

            echo "✅ Production deployment complete!"
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          IMAGE_TAG: ${{ env.IMAGE_TAG }}

      - name: Post-deploy verification
        run: |
          sleep 20
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            https://yourdomain.com/api/health)

          if [ "$STATUS" != "200" ]; then
            echo "❌ Production health check failed!"
            exit 1
          fi
          echo "✅ Production is healthy!"

      - name: Create GitHub deployment
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.repos.createDeploymentStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              deployment_id: context.payload.deployment?.id || 1,
              state: 'success',
              environment_url: 'https://yourdomain.com',
              description: 'Deployed successfully'
            });
```

### Setting Up GitHub Environments

```
In GitHub repo: Settings → Environments

1. Create "staging" environment:
   - No protection rules (auto-deploy)
   - Secrets:
     STAGING_HOST=your-staging-ip
     STAGING_SSH_KEY=your-private-key

2. Create "production" environment:
   - ✅ Required reviewers: add yourself
   - ✅ Wait timer: 5 minutes (optional)
   - Secrets:
     PROD_HOST=your-prod-ip
     PROD_SSH_KEY=your-private-key
     DB_PASSWORD=your-db-password

3. Repository secrets (Settings → Secrets → Actions):
   DOCKERHUB_USERNAME=pramendraatwork
   DOCKERHUB_TOKEN=your-dockerhub-access-token
```

### What You Learned

- ✅ Parallel jobs (backend + frontend tests)
- ✅ Service containers (PostgreSQL in CI)
- ✅ Docker build with layer caching
- ✅ GitHub Environments with approval
- ✅ SSH deployment in pipeline
- ✅ Smoke tests after deploy
- ✅ Job summaries in GitHub

---

## Project 3: Multi-Environment Kubernetes Pipeline 🔴 Advanced

### What You'll Build

A production-grade GitOps pipeline — builds Docker images, pushes to registry, updates Kubernetes manifests, deploys to dev/staging/production clusters, and handles rollback automatically.

```
ADVANCED GITOPS PIPELINE:
──────────────────────────────────────────────────────────────
Push to feature/* branch
        │
        ▼
[CI] lint + test + security scan
        │
Push to develop branch
        │
        ▼
[CD] Build image → push → deploy to DEV k8s
        │
        ▼
[CD] Integration tests on DEV
        │
Push to main branch (PR merged)
        │
        ▼
[CD] Build image with semantic version tag
        │
        ▼
[CD] Update k8s manifests in GitOps repo
        │
        ▼
ArgoCD detects change → deploys to STAGING
        │
        ▼
[CD] E2E tests on STAGING
        │
        ▼
⏸️ Manual approval
        │
        ▼
[CD] Update production manifests
        │
        ▼
ArgoCD deploys to PRODUCTION
        │
        ▼
[CD] Production verification + notify
──────────────────────────────────────────────────────────────
```

### Project Structure

```
k8s-cicd/
├── .github/
│   └── workflows/
│       ├── ci.yml          ← runs on all branches
│       ├── deploy-dev.yml  ← runs on develop
│       └── deploy-prod.yml ← runs on main
├── src/
│   └── app.js
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patches.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patches.yaml
│   └── production/
│       ├── kustomization.yaml
│       └── patches.yaml
├── Dockerfile
└── scripts/
    └── update-image-tag.sh
```

### Kubernetes Base Manifests

```yaml
# k8s/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: pramendraatwork/my-app:latest  # updated by pipeline!
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: '128Mi'
              cpu: '100m'
            limits:
              memory: '256Mi'
              cpu: '500m'
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
          env:
            - name: NODE_ENV
              value: production
            - name: APP_VERSION
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['version']
```

```yaml
# k8s/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

```yaml
# k8s/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: production
bases:
  - ../base
patches:
  - patches.yaml
images:
  - name: pramendraatwork/my-app
    newTag: latest    # pipeline updates this!
```

```yaml
# k8s/production/patches.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5    # more replicas in production!
```

### Image Tag Update Script

```bash
#!/bin/bash
# scripts/update-image-tag.sh
# Updates image tag in kustomization.yaml

set -e

ENVIRONMENT=$1
NEW_TAG=$2
KUSTOMIZATION="k8s/${ENVIRONMENT}/kustomization.yaml"

if [ -z "$ENVIRONMENT" ] || [ -z "$NEW_TAG" ]; then
  echo "Usage: $0 <environment> <new-tag>"
  exit 1
fi

echo "Updating $ENVIRONMENT to tag: $NEW_TAG"

# Update the image tag in kustomization.yaml
sed -i "s|newTag:.*|newTag: $NEW_TAG|" "$KUSTOMIZATION"

echo "Updated $KUSTOMIZATION:"
grep "newTag" "$KUSTOMIZATION"
```

### CI Workflow (All Branches)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: ['**']
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    name: 🔍 CI Checks
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install & Lint
        run: npm ci && npm run lint

      - name: Test with coverage
        run: npm test -- --coverage

      - name: Security scan
        run: npm audit --audit-level=high

      - name: Validate K8s manifests
        uses: azure/k8s-lint@v1
        with:
          manifests: |
            k8s/base/deployment.yaml
            k8s/base/service.yaml

      - name: Validate Dockerfile
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
```

### Deploy to Dev (On Develop Branch Push)

```yaml
# .github/workflows/deploy-dev.yml
name: Deploy to Dev

on:
  push:
    branches: [develop]

env:
  IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/my-app
  IMAGE_TAG: dev-${{ github.sha }}

jobs:
  build:
    name: 🐳 Build & Push
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - uses: docker/setup-buildx-action@v3

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.IMAGE }}:${{ env.IMAGE_TAG }}
            ${{ env.IMAGE }}:dev-latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-dev:
    name: 🚀 Deploy Dev
    runs-on: ubuntu-latest
    needs: build
    environment: dev

    steps:
      - uses: actions/checkout@v4

      - name: Setup kubectl
        uses: azure/setup-kubectl@v3

      - name: Configure kubectl for dev cluster
        run: |
          echo "${{ secrets.DEV_KUBECONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Update image tag
        run: |
          chmod +x scripts/update-image-tag.sh
          ./scripts/update-image-tag.sh dev ${{ env.IMAGE_TAG }}

      - name: Deploy to dev with kustomize
        run: |
          export KUBECONFIG=kubeconfig
          kubectl apply -k k8s/dev/
          kubectl rollout status deployment/my-app -n dev --timeout=3m

      - name: Run integration tests
        run: |
          export KUBECONFIG=kubeconfig

          # Wait for pod to be ready
          kubectl wait pod \
            -l app=my-app \
            -n dev \
            --for=condition=Ready \
            --timeout=60s

          # Port forward and test
          kubectl port-forward \
            service/my-app-service 8080:80 \
            -n dev &
          PF_PID=$!
          sleep 5

          # Run tests
          curl -sf http://localhost:8080/health
          curl -sf http://localhost:8080/api/todos

          kill $PF_PID
          echo "✅ Integration tests passed!"
```

### Deploy to Production (On Main Branch Push)

```yaml
# .github/workflows/deploy-prod.yml
name: Deploy to Production

on:
  push:
    branches: [main]

env:
  IMAGE: ${{ secrets.DOCKERHUB_USERNAME }}/my-app

jobs:
  # ── Semantic versioning ─────────────────────────────────────
  version:
    name: 📦 Get Version
    runs-on: ubuntu-latest
    outputs:
      new-tag: ${{ steps.semver.outputs.new-tag }}

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get next semantic version
        id: semver
        uses: ietf-tools/semver-action@v1
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          branch: main
          patchAll: true

      - name: Show new version
        run: echo "New version: ${{ steps.semver.outputs.new-tag }}"

  # ── Build production image ──────────────────────────────────
  build:
    name: 🐳 Build Production Image
    runs-on: ubuntu-latest
    needs: version

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - uses: docker/setup-buildx-action@v3

      - name: Build & push production image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.IMAGE }}:${{ needs.version.outputs.new-tag }}
            ${{ env.IMAGE }}:latest
          build-args: |
            APP_VERSION=${{ needs.version.outputs.new-tag }}
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
            GIT_COMMIT=${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE }}:${{ needs.version.outputs.new-tag }}
          severity: CRITICAL
          exit-code: 1

  # ── Deploy staging ──────────────────────────────────────────
  deploy-staging:
    name: 🎭 Deploy Staging
    runs-on: ubuntu-latest
    needs: [version, build]
    environment:
      name: staging
      url: https://staging.yourdomain.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl staging
        run: |
          echo "${{ secrets.STAGING_KUBECONFIG }}" | base64 -d > kubeconfig

      - name: Update staging image tag
        run: |
          chmod +x scripts/update-image-tag.sh
          ./scripts/update-image-tag.sh staging ${{ needs.version.outputs.new-tag }}

      - name: Commit manifest update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add k8s/staging/
          git commit -m "ci: update staging to ${{ needs.version.outputs.new-tag }} [skip ci]"
          git push

      - name: Deploy to staging
        run: |
          export KUBECONFIG=kubeconfig
          kubectl apply -k k8s/staging/
          kubectl rollout status deployment/my-app \
            -n staging --timeout=5m

      - name: Run E2E tests
        run: |
          export KUBECONFIG=kubeconfig
          kubectl port-forward \
            service/my-app-service 8080:80 -n staging &
          sleep 10

          # E2E tests
          curl -sf http://localhost:8080/health
          curl -sf http://localhost:8080/api/todos

          echo "✅ E2E tests passed on staging!"

  # ── Approve & deploy production ─────────────────────────────
  deploy-production:
    name: 🎯 Deploy Production
    runs-on: ubuntu-latest
    needs: [version, deploy-staging]
    environment:
      name: production          # ← requires manual approval!
      url: https://yourdomain.com

    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Configure kubectl production
        run: |
          echo "${{ secrets.PROD_KUBECONFIG }}" | base64 -d > kubeconfig

      - name: Update production image tag
        run: |
          chmod +x scripts/update-image-tag.sh
          ./scripts/update-image-tag.sh production \
            ${{ needs.version.outputs.new-tag }}

      - name: Commit manifest update
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add k8s/production/
          git commit -m "ci: deploy ${{ needs.version.outputs.new-tag }} to production [skip ci]"
          git push

      - name: Deploy to production
        run: |
          export KUBECONFIG=kubeconfig
          kubectl apply -k k8s/production/
          kubectl rollout status deployment/my-app \
            -n production --timeout=10m

      - name: Production health verification
        run: |
          sleep 30
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            https://yourdomain.com/api/health)

          if [ "$STATUS" != "200" ]; then
            echo "❌ Production health check failed!"

            # Rollback!
            export KUBECONFIG=kubeconfig
            kubectl rollout undo deployment/my-app -n production
            echo "⏪ Rolled back!"
            exit 1
          fi
          echo "✅ Production is healthy!"

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ needs.version.outputs.new-tag }}
          name: Release ${{ needs.version.outputs.new-tag }}
          generate_release_notes: true
          body: |
            ## 🚀 Deployed to Production

            **Version:** ${{ needs.version.outputs.new-tag }}
            **Image:** ${{ env.IMAGE }}:${{ needs.version.outputs.new-tag }}
            **Commit:** ${{ github.sha }}

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚀 *${{ needs.version.outputs.new-tag }}* deployed to production!\n<https://yourdomain.com|View Site> | <${{ github.event.compare }}|See Changes>"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-deploy.yml
# Can be called from any other workflow!
name: Reusable Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      kubeconfig:
        required: true

jobs:
  deploy:
    name: Deploy to ${{ inputs.environment }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        run: echo "${{ secrets.kubeconfig }}" | base64 -d > kubeconfig

      - name: Deploy
        run: |
          ./scripts/update-image-tag.sh \
            ${{ inputs.environment }} \
            ${{ inputs.image-tag }}
          export KUBECONFIG=kubeconfig
          kubectl apply -k k8s/${{ inputs.environment }}/
          kubectl rollout status deployment/my-app \
            -n ${{ inputs.environment }} --timeout=5m

# Call it from another workflow:
# jobs:
#   deploy-dev:
#     uses: ./.github/workflows/reusable-deploy.yml
#     with:
#       environment: dev
#       image-tag: ${{ needs.build.outputs.tag }}
#     secrets:
#       kubeconfig: ${{ secrets.DEV_KUBECONFIG }}
```

### What You Learned

- ✅ GitOps pattern (Git as source of truth)
- ✅ Semantic versioning in pipeline
- ✅ Kustomize for environment-specific configs
- ✅ Multi-cluster Kubernetes deployments
- ✅ Image vulnerability scanning (Trivy)
- ✅ Automatic rollback on failure
- ✅ GitHub Releases with auto-generated notes
- ✅ Reusable workflows
- ✅ Slack notifications
- ✅ Full GitOps with ArgoCD-ready manifests

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| Node.js CI Pipeline | 🟢 Beginner | Full CI with matrix builds | lint, test, coverage, PR checks |
| Full Stack with Docker | 🟡 Intermediate | CI/CD with Docker & server deploy | parallel jobs, environments, SSH |
| Kubernetes GitOps | 🔴 Advanced | Multi-cluster production pipeline | GitOps, semver, kustomize, ArgoCD |

> 💪 **Challenge**: Start with Project 1 today — it takes less than 30 minutes to set up. Add it to your existing Node.js project on GitHub. Every future push will automatically test your code. That's real CI/CD from day one!