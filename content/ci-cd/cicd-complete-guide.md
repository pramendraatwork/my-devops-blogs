---
title: "CI/CD — Complete Guide from Beginner to Advanced"
date: 2024-02-25
draft: false
description: "Everything about CI/CD: concepts, diagrams, Jenkins pipelines, GitHub Actions, other tools, and real beginner to advanced projects."
categories: ["ci-cd"]
tags: ["ci-cd", "jenkins", "github-actions", "pipeline", "automation", "devops"]
showToc: true
TocOpen: true
---

## 1. What is CI/CD? 🤔

CI/CD stands for **Continuous Integration** and **Continuous Delivery/Deployment**. It's the practice of automating everything between writing code and running it in production.

> 💡 **Simple analogy**: Imagine a factory assembly line. Raw materials go in one end, finished products come out the other — automatically, consistently, every time. CI/CD is that assembly line for your code.

### The Old Way vs The CI/CD Way

```
THE OLD WAY (release hell 😩)
──────────────────────────────────────────────────────────
Developer writes code for 3 months
         │
         ▼
Manually merge everyone's code (conflict nightmare!)
         │
         ▼
Manually run tests (if anyone remembers)
         │
         ▼
Manually build the application
         │
         ▼
Manually deploy (at 2am on a Friday 😭)
         │
         ▼
Something breaks. Rollback manually.
         │
         ▼
Post-mortem meeting. Everyone blames each other. 🔥

THE CI/CD WAY (ship fast, sleep well 😊)
──────────────────────────────────────────────────────────
Developer pushes code
         │
         ▼
Pipeline automatically triggered
         │
         ▼
Code merged → tests run → build created → deployed
         │
         ▼
Team notified ✅ Done in minutes, not months!
```

---

## 2. CI vs CD vs CD 📊

People often confuse the three. Here's the clear difference:

```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD PIPELINE                           │
│                                                             │
│  CODE  →  CI  →  CD (Delivery)  →  CD (Deployment)         │
│                                                             │
│ ┌──────┐  ┌──────────────────┐  ┌──────────┐  ┌─────────┐ │
│ │Write │→ │   Continuous     │→ │Continuous│→ │Continuous│ │
│ │Code  │  │  Integration     │  │ Delivery │  │Deployment│ │
│ └──────┘  └──────────────────┘  └──────────┘  └─────────┘ │
│           • Build              • Deploy to    • Auto deploy │
│           • Unit tests           staging      to production │
│           • Code quality       • Integration  • No human   │
│           • Security scan        tests        approval     │
│           • Merge to main      • Manual gate               │
│                                  to prod                   │
└─────────────────────────────────────────────────────────────┘
```

| Term | Full form | What it does | Human approval? |
|---|---|---|---|
| **CI** | Continuous Integration | Build + test on every commit | No |
| **CD** | Continuous Delivery | Auto deploy to staging | Yes (for prod) |
| **CD** | Continuous Deployment | Auto deploy to production | No |

---

## 3. The Full CI/CD Flow 🔄

```
DEVELOPER MACHINE          PIPELINE                    ENVIRONMENTS
──────────────────────────────────────────────────────────────────

git push ──────────────→ ┌─────────────────────┐
                         │   TRIGGER            │
                         │  (webhook/poll)      │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   SOURCE CONTROL     │
                         │   Checkout code      │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   BUILD              │
                         │  Compile/Install     │
                         │  deps                │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   TEST               │
                         │  Unit tests          │
                         │  Integration tests   │
                         │  Code coverage       │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   CODE QUALITY       │
                         │  SonarQube scan      │
                         │  Security audit      │
                         │  Lint check          │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   PACKAGE            │
                         │  Docker image build  │
                         │  Push to registry    │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
          ┌─────────▼───┐  ┌───────▼─────┐  ┌─────▼──────┐
          │  STAGING     │  │   QA        │  │ PRODUCTION  │
          │  Auto deploy │  │  Auto deploy│  │ Manual gate │
          └─────────────┘  └─────────────┘  └────────────┘

If any stage fails → pipeline stops → developer notified immediately!
```

---

## 4. CI/CD with Jenkins 🤖

Jenkins is the most popular self-hosted CI/CD tool. You manage the server, you control everything.

### Jenkins CI/CD Flow

```
GitHub/GitLab
     │
     │ webhook
     ▼
┌─────────────────────────────────────────────┐
│              JENKINS MASTER                  │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │           PIPELINE STAGES           │   │
│  │                                     │   │
│  │  Checkout → Build → Test → Package  │   │
│  │      → Deploy Staging → Approve     │   │
│  │          → Deploy Production        │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  Distributes work to agents ↓               │
└──────────┬──────────────────────────────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌───────┐    ┌───────┐
│Agent 1│    │Agent 2│
│(Linux)│    │(Docker│
└───────┘    └───────┘
```

### Complete Jenkins CI/CD Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myrepo/my-app'
        DOCKER_TAG   = "${env.BUILD_NUMBER}"
        APP_PORT     = '3000'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('📥 Checkout') {
            steps {
                checkout scm
                sh 'git log --oneline -5'
            }
        }

        stage('📦 Install') {
            steps {
                sh 'npm ci'
            }
        }

        stage('🔍 Lint & Quality') {
            parallel {
                stage('Lint') {
                    steps { sh 'npm run lint' }
                }
                stage('Security Audit') {
                    steps { sh 'npm audit --audit-level moderate' }
                }
            }
        }

        stage('🧪 Test') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('🐳 Build Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('📤 Push Image') {
            when { branch 'main' }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('🚀 Deploy Staging') {
            when { branch 'main' }
            steps {
                sh """
                    docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker stop app-staging || true
                    docker rm app-staging || true
                    docker run -d --name app-staging \
                        -p 3001:${APP_PORT} \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }

        stage('✅ Approve Production') {
            when { branch 'main' }
            input {
                message 'Deploy to production?'
                ok 'Deploy!'
                submitter 'admin'
            }
            steps {
                echo 'Approved! Deploying to production...'
            }
        }

        stage('🎯 Deploy Production') {
            when { branch 'main' }
            steps {
                sh """
                    docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker stop app-prod || true
                    docker rm app-prod || true
                    docker run -d --name app-prod \
                        -p 80:${APP_PORT} \
                        --restart unless-stopped \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded!'
            slackSend color: 'good', message: "✅ ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded!"
        }
        failure {
            echo '❌ Pipeline failed!'
            slackSend color: 'danger', message: "❌ ${env.JOB_NAME} #${env.BUILD_NUMBER} failed!"
        }
        always {
            cleanWs()
        }
    }
}
```

---

## 5. CI/CD with GitHub Actions ⚡

GitHub Actions is cloud-native CI/CD built directly into GitHub. No server to manage — just push code and it runs.

### GitHub Actions Architecture

```
GitHub Repository
       │
       │ on: push / pull_request / schedule
       ▼
┌──────────────────────────────────────────┐
│           GITHUB ACTIONS                  │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │           WORKFLOW (.yml)          │  │
│  │                                    │  │
│  │  ┌───────┐  ┌───────┐  ┌───────┐  │  │
│  │  │ Job 1 │  │ Job 2 │  │ Job 3 │  │  │
│  │  │ test  │→ │ build │→ │deploy │  │  │
│  │  └───────┘  └───────┘  └───────┘  │  │
│  │  runs on    runs on    runs on     │  │
│  │  ubuntu     ubuntu     ubuntu      │  │
│  └────────────────────────────────────┘  │
│                                          │
│  GitHub provides the runners (FREE!)     │
└──────────────────────────────────────────┘
```

### GitHub Actions Key Concepts

```
┌─────────────────────────────────────────────────────────┐
│                GITHUB ACTIONS CONCEPTS                   │
│                                                         │
│  Workflow  = The entire automation (.github/workflows/) │
│  Event     = What triggers it (push, PR, schedule)     │
│  Job       = A group of steps (runs on one machine)    │
│  Step      = Individual task (run command or action)   │
│  Action    = Reusable unit (actions/checkout@v4)       │
│  Runner    = Machine that runs jobs (ubuntu/windows)   │
│  Secret    = Encrypted variable (API keys, passwords)  │
│  Artifact  = Files saved from build (test reports)     │
└─────────────────────────────────────────────────────────┘
```

### Complete GitHub Actions CI/CD Pipeline

```yaml
# .github/workflows/cicd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:    # manual trigger

env:
  DOCKER_IMAGE: myrepo/my-app
  NODE_VERSION: '20'

jobs:
  # ── Job 1: Test ──────────────────────────────────────────
  test:
    name: 🧪 Test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: coverage/
          retention-days: 7

  # ── Job 2: Security Scan ─────────────────────────────────
  security:
    name: 🔒 Security Scan
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Run security audit
        run: npm audit --audit-level high
      - name: OWASP dependency check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'my-app'
          path: '.'
          format: 'HTML'

  # ── Job 3: Build ─────────────────────────────────────────
  build:
    name: 🐳 Build & Push
    runs-on: ubuntu-latest
    needs: [test, security]    # wait for test and security
    if: github.ref == 'refs/heads/main'

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.DOCKER_IMAGE }}
          tags: |
            type=sha,prefix=sha-
            type=raw,value=latest

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ── Job 4: Deploy Staging ────────────────────────────────
  deploy-staging:
    name: 🚀 Deploy Staging
    runs-on: ubuntu-latest
    needs: build
    environment: staging

    steps:
      - name: Deploy to staging
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            docker pull ${{ env.DOCKER_IMAGE }}:latest
            docker stop app-staging || true
            docker rm app-staging || true
            docker run -d --name app-staging \
              -p 3001:3000 \
              ${{ env.DOCKER_IMAGE }}:latest

  # ── Job 5: Deploy Production ─────────────────────────────
  deploy-production:
    name: 🎯 Deploy Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: production    # requires manual approval in GitHub

    steps:
      - name: Deploy to production
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            docker pull ${{ env.DOCKER_IMAGE }}:latest
            docker stop app-prod || true
            docker rm app-prod || true
            docker run -d --name app-prod \
              -p 80:3000 \
              --restart unless-stopped \
              ${{ env.DOCKER_IMAGE }}:latest

      - name: Notify Slack
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "✅ Deployed to production! Build: ${{ github.run_number }}"}
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## 6. Other CI/CD Tools 🛠️

```
CI/CD TOOLS LANDSCAPE
──────────────────────────────────────────────────────────────

CLOUD-NATIVE (hosted — no server management)
┌─────────────────────────────────────────────────────────┐
│ GitHub Actions  │ Built into GitHub. Best for GitHub    │
│                 │ projects. Free for public repos.      │
├─────────────────┼─────────────────────────────────────  │
│ GitLab CI/CD    │ Built into GitLab. YAML-based.        │
│                 │ Great self-hosted option too.          │
├─────────────────┼─────────────────────────────────────  │
│ CircleCI        │ Fast, Docker-first. Great for         │
│                 │ startups. Easy to configure.          │
├─────────────────┼─────────────────────────────────────  │
│ Travis CI       │ One of the originals. Simple.         │
│                 │ Losing popularity to GitHub Actions.  │
├─────────────────┼─────────────────────────────────────  │
│ Bitbucket Pipes │ Built into Bitbucket. Good for        │
│                 │ Atlassian ecosystem (Jira, Confluence) │
└─────────────────────────────────────────────────────────┘

SELF-HOSTED (you manage the server)
┌─────────────────────────────────────────────────────────┐
│ Jenkins         │ Most popular. 1800+ plugins.          │
│                 │ Maximum flexibility. Complex setup.   │
├─────────────────┼─────────────────────────────────────  │
│ GitLab CI       │ Can be self-hosted. Full DevOps       │
│                 │ platform (issues, wiki, registry)     │
├─────────────────┼─────────────────────────────────────  │
│ TeamCity        │ JetBrains product. Great for Java.   │
│                 │ Enterprise-friendly.                  │
├─────────────────┼─────────────────────────────────────  │
│ Drone CI        │ Lightweight, Docker-native.           │
│                 │ Simple YAML config.                   │
└─────────────────────────────────────────────────────────┘

CLOUD PROVIDER CI/CD
┌─────────────────────────────────────────────────────────┐
│ AWS CodePipeline│ Native AWS CI/CD. Works with          │
│ + CodeBuild     │ CodeCommit, ECR, ECS, Lambda.         │
├─────────────────┼─────────────────────────────────────  │
│ Azure DevOps    │ Microsoft's full DevOps platform.     │
│ Pipelines       │ Great for .NET and Azure deployments. │
├─────────────────┼─────────────────────────────────────  │
│ Google Cloud    │ GCP-native. Integrates with GKE,     │
│ Build           │ Cloud Run, Artifact Registry.         │
└─────────────────────────────────────────────────────────┘
```

### Tool Comparison

| Tool | Best for | Difficulty | Cost |
|---|---|---|---|
| GitHub Actions | GitHub projects | ⭐⭐ | Free tier |
| Jenkins | Enterprise, complex | ⭐⭐⭐⭐ | Free (infra cost) |
| GitLab CI | GitLab projects | ⭐⭐ | Free tier |
| CircleCI | Startups | ⭐⭐ | Free tier |
| AWS CodePipeline | AWS workloads | ⭐⭐⭐ | Pay per use |
| Azure DevOps | Microsoft stack | ⭐⭐⭐ | Free tier |

---

## 7. Beginner Project — Your First CI/CD Pipeline 🎯

Let's build a real pipeline step by step. We'll use GitHub Actions (easiest to start).

### Project: Simple Node.js API with CI/CD

```
PROJECT STRUCTURE:
──────────────────────────────────────────
my-first-cicd/
├── .github/
│   └── workflows/
│       └── pipeline.yml    ← our pipeline
├── src/
│   └── app.js              ← our app
├── tests/
│   └── app.test.js         ← our tests
├── Dockerfile              ← containerize
├── package.json
└── README.md
```

**Step 1: Create the app**

```javascript
// src/app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ 
    message: 'Hello from CI/CD!', 
    version: process.env.VERSION || '1.0.0'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

module.exports = app;

if (require.main === module) {
  app.listen(3000, () => console.log('Server running on port 3000'));
}
```

**Step 2: Write tests**

```javascript
// tests/app.test.js
const request = require('supertest');
const app = require('../src/app');

describe('API Tests', () => {
  test('GET / returns hello message', async () => {
    const res = await request(app).get('/');
    expect(res.status).toBe(200);
    expect(res.body.message).toBe('Hello from CI/CD!');
  });

  test('GET /health returns healthy', async () => {
    const res = await request(app).get('/health');
    expect(res.status).toBe(200);
    expect(res.body.status).toBe('healthy');
  });
});
```

**Step 3: Create Dockerfile**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src/ ./src/
EXPOSE 3000
CMD ["node", "src/app.js"]
```

**Step 4: The Pipeline**

```yaml
# .github/workflows/pipeline.yml
name: My First CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # Stage 1: Test
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install
        run: npm ci

      - name: Test
        run: npm test

  # Stage 2: Build (only on main)
  build:
    name: Build Docker Image
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t my-first-app:${{ github.run_number }} .

      - name: Test image runs
        run: |
          docker run -d -p 3000:3000 --name test-app \
            my-first-app:${{ github.run_number }}
          sleep 3
          curl http://localhost:3000/health
          docker stop test-app

  # Stage 3: Done!
  notify:
    name: Notify
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Success message
        run: echo "🎉 Pipeline passed! Build ${{ github.run_number }}"
```

**Step 5: Push and watch it run!**

```bash
git init
git add .
git commit -m "feat: add CI/CD pipeline"
git remote add origin https://github.com/YOUR_USERNAME/my-first-cicd.git
git push -u origin main
# Go to Actions tab and watch the magic! ✨
```

---

## 8. Intermediate Project — Full Stack with Deployment 🏗️

```yaml
# .github/workflows/fullstack.yml
name: Full Stack CI/CD

on:
  push:
    branches: [main]

jobs:
  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: cd frontend && npm ci && npm test

  test-backend:
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
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: cd backend && npm ci
      - run: cd backend && npm test
        env:
          DATABASE_URL: postgres://postgres:testpass@localhost/testdb

  build-and-deploy:
    needs: [test-frontend, test-backend]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build and push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          push: true
          tags: myrepo/frontend:latest

      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          push: true
          tags: myrepo/backend:latest

      - name: Deploy with docker-compose
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ubuntu
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /app
            docker compose pull
            docker compose up -d
            docker compose ps
```

---

## 9. Advanced CI/CD Patterns 🚀

### Blue-Green Deployment

```
BLUE-GREEN DEPLOYMENT
──────────────────────────────────────────────────────────

BEFORE DEPLOY:
Load Balancer ──→ Blue (v1.0 - LIVE) ✅
                  Green (empty)

DURING DEPLOY:
Load Balancer ──→ Blue (v1.0 - LIVE) ✅
                  Green (v2.0 - deploying...)

AFTER DEPLOY:
Load Balancer ──→ Blue (v1.0 - standby)
                  Green (v2.0 - LIVE) ✅

ROLLBACK (instant!):
Load Balancer ──→ Blue (v1.0 - LIVE) ✅  ← just switch back!
                  Green (v2.0 - standby)
```

### Canary Deployment

```
CANARY DEPLOYMENT
──────────────────────────────────────────────────────────
Start: 100% traffic to v1.0

Step 1: 5% → v2.0,  95% → v1.0  (watch metrics)
Step 2: 25% → v2.0, 75% → v1.0  (still good?)
Step 3: 50% → v2.0, 50% → v1.0  (looking great!)
Step 4: 100% → v2.0             (full rollout!)

If metrics go bad at any step → rollback instantly!
```

### GitOps Pattern

```yaml
# GitOps: Git is the single source of truth
# ArgoCD watches your Git repo and syncs to Kubernetes

# 1. Developer pushes code
# 2. CI pipeline builds and pushes Docker image
# 3. CI pipeline updates image tag in Git repo
# 4. ArgoCD detects Git change
# 5. ArgoCD syncs Kubernetes cluster to match Git
# 6. Zero manual kubectl commands needed!

# Update image tag step in GitHub Actions:
- name: Update image tag in GitOps repo
  run: |
    git clone https://github.com/org/k8s-configs.git
    cd k8s-configs
    sed -i "s|image: myapp:.*|image: myapp:${{ github.sha }}|" \
      deployments/myapp.yaml
    git commit -am "ci: update myapp image to ${{ github.sha }}"
    git push
```

---

## 10. CI/CD Best Practices ✅

```
DO ✅                                  DON'T ❌
──────────────────────────────────────────────────────────
Fail fast — test first                 Run deploy before tests
Keep pipelines under 10 minutes        Let pipelines run for hours
Store secrets in vault/secrets         Hardcode passwords in YAML
Test in production-like env            Only test locally
Use pipeline as code (YAML/Jenkinsfile) Click through UI to configure
Pin action/plugin versions             Always use @latest
Notify team on failures               Silently fail
Have rollback plan ready              YOLO deploy with no rollback
Deploy small changes often            Big bang deployments monthly
Monitor after every deploy            Deploy and forget
```

---

## 11. CI/CD Cheatsheet 📋

```bash
# GITHUB ACTIONS TRIGGERS
on: push                    # any push
on: pull_request            # any PR
on: schedule                # cron schedule
on: workflow_dispatch       # manual trigger
on: release                 # when release created

# COMMON ACTIONS
actions/checkout@v4         # checkout code
actions/setup-node@v4       # setup Node.js
actions/setup-python@v5     # setup Python
actions/cache@v4            # cache dependencies
actions/upload-artifact@v4  # save build output
docker/build-push-action@v5 # build & push Docker

# GITHUB ACTIONS CONTEXT
github.sha                  # commit hash
github.ref                  # branch/tag ref
github.run_number           # build number
github.actor                # who triggered
secrets.MY_SECRET           # access secrets
env.MY_VAR                  # environment variable

# JENKINS PIPELINE SNIPPETS
agent any                   # run anywhere
agent { label 'linux' }     # specific agent
when { branch 'main' }      # conditional
parallel { stage()... }     # run in parallel
input { message 'Approve?' }# manual gate
cleanWs()                   # clean workspace
env.BUILD_NUMBER            # build number
env.WORKSPACE               # build directory
```

---

## What's Next? 🚀

CI/CD is the glue that holds DevOps together. Now connect it with:

- **Docker** — CI/CD builds Docker images automatically
- **Kubernetes** — CD deploys to K8s clusters
- **Monitoring** — detect issues right after deployment
- **AWS** — CodePipeline, CodeBuild, CodeDeploy for AWS-native CI/CD

> 💪 **Challenge**: Take your existing GitHub project and add a GitHub Actions workflow today. Start with just one job that runs your tests. Add more stages gradually. That's how every DevOps engineer learns — one pipeline at a time!