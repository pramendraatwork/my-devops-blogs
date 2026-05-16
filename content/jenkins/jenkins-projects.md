---
title: "Jenkins Projects — Beginner to Advanced (3 Real Projects)"
date: 2024-03-22
draft: false
description: "3 hands-on Jenkins projects: beginner first pipeline, intermediate Node.js CI/CD, and advanced multi-environment deployment with Docker and notifications."
categories: ["jenkins"]
tags: ["jenkins", "ci-cd", "pipeline", "projects", "devops", "automation"]
showToc: true
TocOpen: true
---

## Why These Projects? 🎯

Jenkins is the most used CI/CD tool in enterprise DevOps. These 3 projects take you from running your very first pipeline to building a production-grade multi-environment deployment system.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → Your First Jenkins Pipeline
Project 2 (Intermediate) → Node.js App CI/CD with Testing
Project 3 (Advanced)     → Multi-Environment Docker Deployment
──────────────────────────────────────────────────────────────
```

---

## Project 1: Your First Jenkins Pipeline 🟢 Beginner

### What You'll Build

Install Jenkins, create your first pipeline, run basic stages, and understand how Jenkins works. By the end you will have a working pipeline that checks out code, builds, and runs tests.

```
PIPELINE STAGES:
──────────────────────────────────────────────────────────────
Stage 1: Checkout    → Pull code from GitHub
Stage 2: Build       → Install dependencies
Stage 3: Test        → Run tests
Stage 4: Report      → Show results
──────────────────────────────────────────────────────────────

Pipeline View in Jenkins:
✅ Checkout  →  ✅ Build  →  ✅ Test  →  ✅ Report
   2s              8s          12s          1s
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- Installing Jenkins with Docker
- Creating Freestyle and Pipeline jobs
- Basic Jenkinsfile syntax
- Reading pipeline logs
- Triggering builds manually

### Step 1: Install Jenkins with Docker

```bash
# Create Jenkins home directory
mkdir -p ~/jenkins-home

# Run Jenkins with Docker
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v ~/jenkins-home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

# Watch Jenkins start up
docker logs -f jenkins

# Get the initial admin password
docker exec jenkins \
  cat /var/jenkins_home/secrets/initialAdminPassword
```

### Step 2: Initial Setup

```
1. Open browser: http://localhost:8080
2. Paste the initial admin password
3. Click "Install suggested plugins"
4. Wait for plugins to install (~3 minutes)
5. Create admin user:
   - Username: admin
   - Password: admin123
   - Full name: Pramendra Rajput
   - Email: pramendraatwork@gmail.com
6. Keep Jenkins URL as default
7. Click "Start using Jenkins"
```

### Step 3: Create a Simple App

```bash
# Create a simple Node.js app on GitHub
mkdir jenkins-first-pipeline
cd jenkins-first-pipeline
git init && git checkout -b main

# Create app
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Jenkins Pipeline!', status: 'ok' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

module.exports = app;

if (require.main === module) {
  app.listen(3000, () => console.log('Server running on port 3000'));
}
EOF

# Create package.json
cat > package.json << 'EOF'
{
  "name": "jenkins-first-pipeline",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js",
    "test": "node test.js"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

# Create simple test
cat > test.js << 'EOF'
const http = require('http');

// Simple test without framework
function test(name, fn) {
  try {
    fn();
    console.log(`✅ PASS: ${name}`);
  } catch (e) {
    console.error(`❌ FAIL: ${name} - ${e.message}`);
    process.exit(1);
  }
}

function assert(condition, message) {
  if (!condition) throw new Error(message);
}

// Tests
test('App module loads', () => {
  const app = require('./app');
  assert(app, 'App should be defined');
});

test('Environment check', () => {
  const node_version = process.version;
  assert(node_version, 'Node version should be defined');
  console.log(`  Node version: ${node_version}`);
});

test('Package.json valid', () => {
  const pkg = require('./package.json');
  assert(pkg.name, 'Package name required');
  assert(pkg.version, 'Package version required');
  console.log(`  Package: ${pkg.name}@${pkg.version}`);
});

console.log('\n🎉 All tests passed!');
EOF

# Push to GitHub
git add .
git commit -m "feat: initial app for Jenkins pipeline"
git remote add origin https://github.com/pramendraatwork/jenkins-first-pipeline.git
git push -u origin main
```

### Step 4: Create Your First Jenkinsfile

```groovy
// Jenkinsfile — add this to your repo root
pipeline {
    agent any

    // Environment variables
    environment {
        APP_NAME    = 'jenkins-first-pipeline'
        NODE_VERSION = '20'
    }

    // Options
    options {
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timestamps()
    }

    stages {

        // ── Stage 1: Checkout ─────────────────────────────
        stage('📥 Checkout') {
            steps {
                echo '==================================='
                echo "Building: ${APP_NAME}"
                echo "Branch:   ${env.BRANCH_NAME ?: 'main'}"
                echo "Build #:  ${env.BUILD_NUMBER}"
                echo '==================================='
                checkout scm
            }
        }

        // ── Stage 2: Environment Info ─────────────────────
        stage('🔍 Environment') {
            steps {
                sh '''
                    echo "=== System Info ==="
                    echo "OS: $(uname -a)"
                    echo "Node: $(node --version 2>/dev/null || echo 'not installed')"
                    echo "npm:  $(npm --version 2>/dev/null || echo 'not installed')"
                    echo "Git:  $(git --version)"
                    echo "Disk: $(df -h / | tail -1)"
                    echo "RAM:  $(free -h | grep Mem)"
                '''
            }
        }

        // ── Stage 3: Install ──────────────────────────────
        stage('📦 Install') {
            steps {
                echo 'Installing dependencies...'
                sh 'npm install'
                echo 'Dependencies installed!'
            }
        }

        // ── Stage 4: Test ─────────────────────────────────
        stage('🧪 Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
            post {
                success {
                    echo '✅ All tests passed!'
                }
                failure {
                    echo '❌ Tests failed!'
                }
            }
        }

        // ── Stage 5: Report ───────────────────────────────
        stage('📊 Report') {
            steps {
                sh '''
                    echo "=== BUILD REPORT ==="
                    echo "App:      ${APP_NAME}"
                    echo "Build #:  ${BUILD_NUMBER}"
                    echo "Status:   SUCCESS"
                    echo "Time:     $(date)"
                    echo "Workspace: ${WORKSPACE}"
                    echo "===================="
                '''
            }
        }
    }

    // Post actions — always run
    post {
        always {
            echo "Pipeline finished: ${currentBuild.result ?: 'SUCCESS'}"
            cleanWs()    // clean workspace
        }
        success {
            echo '🎉 Pipeline succeeded!'
        }
        failure {
            echo '💥 Pipeline failed! Check logs above.'
        }
    }
}
```

### Step 5: Connect Jenkins to GitHub

```
1. In Jenkins: New Item
2. Name: first-pipeline
3. Type: Pipeline
4. Click OK

5. In Pipeline section:
   - Definition: Pipeline script from SCM
   - SCM: Git
   - Repository URL: https://github.com/pramendraatwork/jenkins-first-pipeline.git
   - Branch: */main
   - Script Path: Jenkinsfile

6. Click Save
7. Click "Build Now"
8. Watch it run! Click the build → Console Output
```

### Step 6: Add a Webhook (Auto Trigger)

```
In GitHub repo:
Settings → Webhooks → Add webhook
  Payload URL: http://YOUR_IP:8080/github-webhook/
  Content type: application/json
  Events: Just the push event
  Active: ✅

In Jenkins job:
Configure → Build Triggers
  ✅ GitHub hook trigger for GITScm polling

Now every git push triggers the pipeline automatically!
```

### Step 7: Read the Results

```
In Jenkins Dashboard you'll see:
✅ Blue circle  = Success
❌ Red circle   = Failed
🔵 Blue ball    = Running
⚪ Grey circle  = Not run yet

Pipeline view shows each stage:
📥 Checkout | 🔍 Env | 📦 Install | 🧪 Test | 📊 Report
    ✅             ✅        ✅           ✅          ✅
    2s             1s        8s           3s          1s
```

### What You Learned

- ✅ Jenkins installation with Docker
- ✅ Pipeline basics (stages, steps, post)
- ✅ Connecting GitHub to Jenkins
- ✅ Webhooks for auto-triggering
- ✅ Reading build logs and results

---

## Project 2: Node.js App CI/CD Pipeline 🟡 Intermediate

### What You'll Build

A complete CI/CD pipeline for a Node.js REST API — linting, testing with coverage, Docker build, Docker Hub push, and deployment to a staging server.

```
FULL PIPELINE:
──────────────────────────────────────────────────────────────
Push to GitHub
      │
      ▼
Jenkins triggered (webhook)
      │
      ▼
✅ Checkout → ✅ Install → ✅ Lint → ✅ Test
      │
      ▼
✅ Coverage Check (must be >70%)
      │
      ▼
✅ Docker Build → ✅ Docker Push
      │
      ▼
✅ Deploy to Staging
      │
      ▼
✅ Health Check → ✅ Notify Team
──────────────────────────────────────────────────────────────
```

### Project Structure

```
nodejs-cicd/
├── src/
│   ├── app.js
│   ├── routes/
│   │   ├── users.js
│   │   └── todos.js
│   └── middleware/
│       └── logger.js
├── tests/
│   ├── users.test.js
│   └── todos.test.js
├── Dockerfile
├── .dockerignore
├── .eslintrc.json
├── Jenkinsfile
└── package.json
```

### The Application

```javascript
// src/app.js
const express = require('express');
const app = express();

app.use(express.json());
app.use(require('./middleware/logger'));

// Routes
app.use('/api/users', require('./routes/users'));
app.use('/api/todos', require('./routes/todos'));

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    version: process.env.APP_VERSION || '1.0.0',
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: 'Not found' });
});

// Error handler
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Internal server error' });
});

module.exports = app;

if (require.main === module) {
  const PORT = process.env.PORT || 3000;
  app.listen(PORT, () => {
    console.log(`🚀 Server running on port ${PORT}`);
  });
}
```

```javascript
// src/routes/users.js
const express = require('express');
const router = express.Router();

let users = [
  { id: 1, name: 'Pramendra Rajput', email: 'pramendraatwork@gmail.com', role: 'admin' },
  { id: 2, name: 'John Doe', email: 'john@example.com', role: 'user' }
];

router.get('/', (req, res) => res.json(users));

router.get('/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'User not found' });
  res.json(user);
});

router.post('/', (req, res) => {
  const { name, email, role = 'user' } = req.body;
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email required' });
  }
  const user = { id: Date.now(), name, email, role };
  users.push(user);
  res.status(201).json(user);
});

router.delete('/:id', (req, res) => {
  const index = users.findIndex(u => u.id === parseInt(req.params.id));
  if (index === -1) return res.status(404).json({ error: 'User not found' });
  users.splice(index, 1);
  res.json({ message: 'User deleted' });
});

module.exports = router;
```

```javascript
// src/middleware/logger.js
module.exports = (req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.path} ${res.statusCode} ${duration}ms`);
  });
  next();
};
```

### Tests

```javascript
// tests/users.test.js
const request = require('supertest');
const app = require('../src/app');

describe('Users API', () => {
  test('GET /api/users returns user list', async () => {
    const res = await request(app).get('/api/users');
    expect(res.status).toBe(200);
    expect(Array.isArray(res.body)).toBe(true);
    expect(res.body.length).toBeGreaterThan(0);
  });

  test('GET /api/users/:id returns user', async () => {
    const res = await request(app).get('/api/users/1');
    expect(res.status).toBe(200);
    expect(res.body.id).toBe(1);
    expect(res.body.name).toBeDefined();
  });

  test('GET /api/users/:id returns 404 for unknown user', async () => {
    const res = await request(app).get('/api/users/9999');
    expect(res.status).toBe(404);
  });

  test('POST /api/users creates user', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: 'Test User', email: 'test@example.com' });
    expect(res.status).toBe(201);
    expect(res.body.name).toBe('Test User');
    expect(res.body.id).toBeDefined();
  });

  test('POST /api/users requires name and email', async () => {
    const res = await request(app)
      .post('/api/users')
      .send({ name: 'No Email' });
    expect(res.status).toBe(400);
  });

  test('GET /health returns healthy', async () => {
    const res = await request(app).get('/health');
    expect(res.status).toBe(200);
    expect(res.body.status).toBe('healthy');
  });
});
```

### Dockerfile

```dockerfile
# Dockerfile
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

### ESLint Config

```json
{
  "env": {
    "node": true,
    "es2021": true,
    "jest": true
  },
  "extends": "eslint:recommended",
  "rules": {
    "no-unused-vars": "warn",
    "no-console": "off",
    "semi": ["error", "always"],
    "quotes": ["error", "single"]
  }
}
```

### The Jenkinsfile

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        APP_NAME     = 'nodejs-cicd'
        DOCKER_IMAGE = "pramendraatwork/${APP_NAME}"
        DOCKER_TAG   = "${env.BUILD_NUMBER}"
        STAGING_HOST = credentials('staging-host')
        STAGING_USER = 'deploy'
        COVERAGE_MIN = '70'
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
        timestamps()
    }

    tools {
        nodejs 'NodeJS-20'    // configure in Jenkins → Tools
    }

    stages {

        // ── Checkout ──────────────────────────────────────
        stage('📥 Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_MSG = sh(
                        script: 'git log -1 --pretty=%s',
                        returnStdout: true
                    ).trim()
                    env.GIT_AUTHOR = sh(
                        script: 'git log -1 --pretty=%an',
                        returnStdout: true
                    ).trim()
                }
                echo "Commit: ${env.GIT_COMMIT_MSG}"
                echo "Author: ${env.GIT_AUTHOR}"
            }
        }

        // ── Install ───────────────────────────────────────
        stage('📦 Install') {
            steps {
                sh 'npm ci'
                sh 'npm list --depth=0'
            }
        }

        // ── Lint & Quality ────────────────────────────────
        stage('🔍 Lint') {
            steps {
                sh 'npm run lint'
            }
            post {
                failure {
                    echo '❌ Linting failed! Fix code style issues.'
                }
            }
        }

        // ── Test with Coverage ────────────────────────────
        stage('🧪 Test') {
            steps {
                sh 'npm test -- --coverage --coverageReporters=text --coverageReporters=lcov'
            }
            post {
                always {
                    // Publish coverage report
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage/lcov-report',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        // ── Coverage Gate ─────────────────────────────────
        stage('📊 Coverage Gate') {
            steps {
                script {
                    def coverage = sh(
                        script: """
                            cat coverage/coverage-summary.json | \
                            node -e "
                                const data = JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));
                                console.log(Math.round(data.total.lines.pct));
                            "
                        """,
                        returnStdout: true
                    ).trim().toInteger()

                    echo "Code coverage: ${coverage}%"
                    echo "Minimum required: ${COVERAGE_MIN}%"

                    if (coverage < COVERAGE_MIN.toInteger()) {
                        error("Coverage ${coverage}% is below minimum ${COVERAGE_MIN}%!")
                    }
                    echo "✅ Coverage gate passed: ${coverage}%"
                }
            }
        }

        // ── Docker Build ──────────────────────────────────
        stage('🐳 Docker Build') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    docker.build("${DOCKER_IMAGE}:latest")
                }
                echo "Built: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }

        // ── Docker Push ───────────────────────────────────
        stage('📤 Docker Push') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com',
                                        'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
                echo "Pushed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }

        // ── Deploy Staging ────────────────────────────────
        stage('🚀 Deploy Staging') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['staging-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no \
                            ${STAGING_USER}@${STAGING_HOST} '
                            echo "Deploying ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop ${APP_NAME} || true
                            docker rm ${APP_NAME} || true
                            docker run -d \
                                --name ${APP_NAME} \
                                --restart unless-stopped \
                                -p 3000:3000 \
                                -e APP_VERSION=${DOCKER_TAG} \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                            echo "Deploy complete!"
                        '
                    """
                }
            }
        }

        // ── Health Check ──────────────────────────────────
        stage('🏥 Health Check') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Wait for app to start
                    sleep(10)

                    def response = sh(
                        script: """
                            curl -s -o /dev/null -w "%{http_code}" \
                            http://${STAGING_HOST}:3000/health
                        """,
                        returnStdout: true
                    ).trim()

                    if (response != '200') {
                        error("Health check failed! Got: ${response}")
                    }
                    echo "✅ App is healthy on staging!"
                }
            }
        }
    }

    // ── Post Actions ──────────────────────────────────────
    post {
        success {
            echo """
            ✅ BUILD SUCCESS
            ═══════════════════════════
            App:     ${APP_NAME}
            Build:   #${BUILD_NUMBER}
            Image:   ${DOCKER_IMAGE}:${DOCKER_TAG}
            Author:  ${GIT_AUTHOR}
            Commit:  ${GIT_COMMIT_MSG}
            ═══════════════════════════
            """
            // Uncomment for Slack:
            // slackSend color: 'good',
            //   message: "✅ ${APP_NAME} #${BUILD_NUMBER} deployed!"
        }
        failure {
            echo """
            ❌ BUILD FAILED
            ═══════════════════════════
            App:    ${APP_NAME}
            Build:  #${BUILD_NUMBER}
            Check logs for details.
            ═══════════════════════════
            """
        }
        always {
            cleanWs()
        }
    }
}
```

### Jenkins Configuration

```
1. Install plugins:
   Manage Jenkins → Plugins → Available
   ✅ NodeJS Plugin
   ✅ Docker Pipeline
   ✅ SSH Agent
   ✅ HTML Publisher
   ✅ Credentials Binding

2. Configure NodeJS:
   Manage Jenkins → Tools → NodeJS
   Name: NodeJS-20
   Version: 20.x

3. Add credentials:
   Manage Jenkins → Credentials → Global → Add
   - ID: dockerhub-credentials
     Type: Username/Password
     Username: pramendraatwork
     Password: your-dockerhub-token

   - ID: staging-ssh-key
     Type: SSH Username with private key
     Username: deploy
     Key: paste private key

   - ID: staging-host
     Type: Secret text
     Value: your-staging-server-ip
```

### What You Learned

- ✅ Multi-stage pipeline with real app
- ✅ Code linting in CI
- ✅ Test coverage gates
- ✅ Docker build and push
- ✅ SSH deployment
- ✅ Health checks after deploy
- ✅ Jenkins credentials management

---

## Project 3: Multi-Environment Docker Deployment 🔴 Advanced

### What You'll Build

A production-grade Jenkins pipeline that deploys to three environments (dev, staging, production) with manual approval gates, automatic rollback on failure, Slack notifications, and parallel testing.

```
ADVANCED PIPELINE FLOW:
──────────────────────────────────────────────────────────────
Push to any branch
        │
        ▼
Parallel: [Lint] [Unit Tests] [Security Scan]
        │
        ▼ (if main branch)
Build Docker Image + Push to Registry
        │
        ▼
Auto Deploy → DEV environment
        │
        ▼
Integration Tests on DEV
        │
        ▼
Auto Deploy → STAGING environment
        │
        ▼
Smoke Tests on STAGING
        │
        ▼
⏸️  MANUAL APPROVAL GATE ← team lead approves
        │
        ▼
Deploy → PRODUCTION (blue-green)
        │
        ▼
Production Health Check
        │           │
     ✅ Pass     ❌ Fail
        │           │
    Notify       Auto Rollback
    Success      → Notify Failure
──────────────────────────────────────────────────────────────
```

### The Advanced Jenkinsfile

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        APP_NAME        = 'my-production-app'
        DOCKER_REGISTRY = 'registry.hub.docker.com'
        DOCKER_IMAGE    = "pramendraatwork/${APP_NAME}"
        DOCKER_TAG      = "${env.BUILD_NUMBER}-${env.GIT_COMMIT?.take(7) ?: 'unknown'}"

        // Environment hosts
        DEV_HOST        = credentials('dev-host')
        STAGING_HOST    = credentials('staging-host')
        PROD_HOST       = credentials('prod-host')

        // Notification
        SLACK_CHANNEL   = '#deployments'
    }

    options {
        timeout(time: 45, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timestamps()
        ansiColor('xterm')
    }

    stages {

        // ── Stage 1: Checkout & Info ──────────────────────
        stage('📥 Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.GIT_COMMIT_MSG = sh(
                        script: 'git log -1 --pretty=%s',
                        returnStdout: true
                    ).trim()
                    env.GIT_AUTHOR = sh(
                        script: 'git log -1 --pretty=%an',
                        returnStdout: true
                    ).trim()

                    notifySlack('started', '🔵')
                }
            }
        }

        // ── Stage 2: Parallel Quality Checks ─────────────
        stage('🔍 Quality Checks') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm ci && npm run lint'
                    }
                }
                stage('Unit Tests') {
                    steps {
                        sh 'npm test -- --coverage'
                    }
                    post {
                        always {
                            junit 'test-results/*.xml'
                        }
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh '''
                            npm audit --audit-level=high
                            echo "Security scan complete"
                        '''
                    }
                }
                stage('Dependency Check') {
                    steps {
                        sh '''
                            npx license-checker --onlyAllow \
                              "MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC" \
                            || echo "License check complete"
                        '''
                    }
                }
            }
        }

        // ── Stage 3: Build Docker Image ───────────────────
        stage('🐳 Build Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    sh """
                        docker build \
                            --build-arg BUILD_NUMBER=${BUILD_NUMBER} \
                            --build-arg GIT_COMMIT=${GIT_COMMIT_SHORT} \
                            --build-arg BUILD_DATE=\$(date -u +%Y-%m-%dT%H:%M:%SZ) \
                            -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                            -t ${DOCKER_IMAGE}:latest \
                            .
                    """
                }
            }
        }

        // ── Stage 4: Push to Registry ─────────────────────
        stage('📤 Push Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    docker.withRegistry("https://${DOCKER_REGISTRY}",
                                        'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()

                        // Tag with branch name too
                        def branchTag = env.BRANCH_NAME.replaceAll('/', '-')
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push(branchTag)

                        if (env.BRANCH_NAME == 'main') {
                            docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                        }
                    }
                }
            }
        }

        // ── Stage 5: Deploy to DEV ────────────────────────
        stage('🔧 Deploy DEV') {
            when { branch 'develop' }
            steps {
                script {
                    deployToEnvironment('dev', DEV_HOST, '3001')
                    sleep(15)
                    healthCheck(DEV_HOST, '3001', 'dev')
                }
            }
        }

        // ── Stage 6: Deploy to STAGING ────────────────────
        stage('🎭 Deploy STAGING') {
            when { branch 'main' }
            steps {
                script {
                    deployToEnvironment('staging', STAGING_HOST, '3002')
                    sleep(15)
                    healthCheck(STAGING_HOST, '3002', 'staging')
                    integrationTests(STAGING_HOST, '3002')
                }
            }
        }

        // ── Stage 7: Manual Approval ──────────────────────
        stage('✋ Approve Production') {
            when { branch 'main' }
            options {
                timeout(time: 24, unit: 'HOURS')
            }
            steps {
                script {
                    notifySlack('approval_needed', '⏸️')

                    input(
                        message: """Deploy ${APP_NAME}:${DOCKER_TAG} to PRODUCTION?
                        
                        Staging verified: ✅
                        Tests passed:     ✅
                        Image:            ${DOCKER_IMAGE}:${DOCKER_TAG}
                        Commit:           ${GIT_COMMIT_MSG}
                        Author:           ${GIT_AUTHOR}
                        """,
                        ok: '🚀 Deploy to Production!',
                        submitter: 'admin,devops-lead',
                        parameters: [
                            choice(
                                name: 'DEPLOY_STRATEGY',
                                choices: ['rolling', 'blue-green'],
                                description: 'Deployment strategy'
                            )
                        ]
                    )
                }
            }
        }

        // ── Stage 8: Deploy to PRODUCTION ─────────────────
        stage('🎯 Deploy PRODUCTION') {
            when { branch 'main' }
            steps {
                script {
                    // Save current version for rollback
                    env.PREVIOUS_TAG = sh(
                        script: """
                            docker ps --filter name=${APP_NAME}-prod \
                            --format '{{.Image}}' | \
                            head -1 | cut -d: -f2
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Previous version: ${env.PREVIOUS_TAG ?: 'none'}"
                    echo "Deploying: ${DOCKER_TAG}"

                    try {
                        blueGreenDeploy(PROD_HOST, '80')
                        sleep(20)
                        healthCheck(PROD_HOST, '80', 'production')
                        notifySlack('success', '✅')
                    } catch (Exception e) {
                        echo "❌ Production deployment failed!"
                        echo "🔄 Rolling back to: ${env.PREVIOUS_TAG}"
                        rollback(PROD_HOST, '80')
                        notifySlack('rollback', '⏪')
                        error("Deployment failed and rolled back: ${e.message}")
                    }
                }
            }
        }
    }

    post {
        success {
            notifySlack('success', '✅')
        }
        failure {
            notifySlack('failure', '❌')
        }
        always {
            cleanWs()
        }
    }
}

// ── Helper Functions ──────────────────────────────────────────

def deployToEnvironment(String env, String host, String port) {
    sshagent(['deploy-ssh-key']) {
        sh """
            ssh -o StrictHostKeyChecking=no deploy@${host} '
                docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                docker stop ${APP_NAME}-${env} || true
                docker rm ${APP_NAME}-${env} || true
                docker run -d \
                    --name ${APP_NAME}-${env} \
                    --restart unless-stopped \
                    -p ${port}:3000 \
                    -e NODE_ENV=${env} \
                    -e APP_VERSION=${DOCKER_TAG} \
                    ${DOCKER_IMAGE}:${DOCKER_TAG}
                echo "Deployed to ${env}!"
            '
        """
    }
    echo "✅ Deployed to ${env}: ${host}:${port}"
}

def blueGreenDeploy(String host, String port) {
    sshagent(['deploy-ssh-key']) {
        sh """
            ssh -o StrictHostKeyChecking=no deploy@${host} '
                # Start GREEN (new version)
                docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                docker run -d \
                    --name ${APP_NAME}-green \
                    -p 3001:3000 \
                    -e NODE_ENV=production \
                    -e APP_VERSION=${DOCKER_TAG} \
                    ${DOCKER_IMAGE}:${DOCKER_TAG}

                sleep 10

                # Health check GREEN
                curl -sf http://localhost:3001/health || exit 1

                # Switch traffic: stop BLUE, rename GREEN → BLUE
                docker stop ${APP_NAME}-blue || true
                docker rm ${APP_NAME}-blue || true
                docker stop ${APP_NAME}-green
                docker rename ${APP_NAME}-green ${APP_NAME}-blue
                docker start ${APP_NAME}-blue

                # Update nginx to point to new port if needed
                echo "Blue-green switch complete!"
            '
        """
    }
}

def healthCheck(String host, String port, String environment) {
    def maxRetries = 5
    def retryInterval = 10

    for (int i = 1; i <= maxRetries; i++) {
        try {
            def response = sh(
                script: """
                    curl -sf -o /dev/null -w "%{http_code}" \
                    http://${host}:${port}/health
                """,
                returnStdout: true
            ).trim()

            if (response == '200') {
                echo "✅ Health check passed on ${environment} (attempt ${i})"
                return
            }
            echo "⚠️  Health check attempt ${i}/${maxRetries}: got ${response}"
        } catch (Exception e) {
            echo "⚠️  Health check attempt ${i}/${maxRetries} failed: ${e.message}"
        }

        if (i < maxRetries) sleep(retryInterval)
    }

    error("Health check failed on ${environment} after ${maxRetries} attempts!")
}

def integrationTests(String host, String port) {
    sh """
        echo "Running integration tests against http://${host}:${port}"
        curl -sf http://${host}:${port}/health
        curl -sf http://${host}:${port}/api/users
        curl -sf -X POST http://${host}:${port}/api/users \
            -H 'Content-Type: application/json' \
            -d '{"name":"Test","email":"test@test.com"}'
        echo "✅ Integration tests passed!"
    """
}

def rollback(String host, String port) {
    if (!env.PREVIOUS_TAG) {
        echo "No previous version to rollback to!"
        return
    }

    sshagent(['deploy-ssh-key']) {
        sh """
            ssh -o StrictHostKeyChecking=no deploy@${host} '
                docker pull ${DOCKER_IMAGE}:${PREVIOUS_TAG}
                docker stop ${APP_NAME}-blue || true
                docker rm ${APP_NAME}-blue || true
                docker run -d \
                    --name ${APP_NAME}-blue \
                    --restart unless-stopped \
                    -p ${port}:3000 \
                    -e NODE_ENV=production \
                    ${DOCKER_IMAGE}:${PREVIOUS_TAG}
                echo "Rolled back to ${PREVIOUS_TAG}!"
            '
        """
    }
}

def notifySlack(String status, String emoji) {
    def messages = [
        'started':          "${emoji} *${APP_NAME}* build #${BUILD_NUMBER} started\nCommit: ${GIT_COMMIT_MSG}\nAuthor: ${GIT_AUTHOR}",
        'success':          "${emoji} *${APP_NAME}* #${BUILD_NUMBER} deployed successfully!\nImage: ${DOCKER_IMAGE}:${DOCKER_TAG}",
        'failure':          "${emoji} *${APP_NAME}* #${BUILD_NUMBER} FAILED!\nCheck: ${BUILD_URL}",
        'rollback':         "${emoji} *${APP_NAME}* rolled back to ${PREVIOUS_TAG}",
        'approval_needed':  "${emoji} *${APP_NAME}* needs approval for production!\nApprove: ${BUILD_URL}input"
    ]

    // Uncomment when Slack plugin installed:
    // slackSend(
    //     channel: SLACK_CHANNEL,
    //     color: status == 'success' ? 'good' : status == 'failure' ? 'danger' : 'warning',
    //     message: messages[status]
    // )
    echo "NOTIFY [${status}]: ${messages[status]}"
}
```

### Shared Library Setup

```groovy
// Create a Jenkins Shared Library for reuse across all pipelines
// In GitHub: jenkins-shared-library/vars/deployApp.groovy

def call(Map config) {
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh "docker build -t ${config.image}:${BUILD_NUMBER} ."
                }
            }
            stage('Test') {
                steps {
                    sh config.testCommand ?: 'npm test'
                }
            }
            stage('Deploy') {
                steps {
                    sh "docker push ${config.image}:${BUILD_NUMBER}"
                }
            }
        }
    }
}

// Use in any Jenkinsfile with just 3 lines:
// @Library('jenkins-shared-library') _
// deployApp(
//     image: 'pramendraatwork/my-app',
//     testCommand: 'npm test'
// )
```

### What You Learned

- ✅ Parallel stages for faster pipelines
- ✅ Multi-environment deployments (dev/staging/prod)
- ✅ Manual approval gates
- ✅ Blue-green deployment strategy
- ✅ Automatic rollback on failure
- ✅ Slack notifications
- ✅ Jenkins shared libraries
- ✅ Helper functions in Jenkinsfile
- ✅ Health check with retries
- ✅ Integration tests in pipeline

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| First Pipeline | 🟢 Beginner | Basic Jenkins pipeline | stages, webhooks, logs |
| Node.js CI/CD | 🟡 Intermediate | Full CI/CD with Docker | testing, coverage, deploy |
| Multi-Env Deploy | 🔴 Advanced | Production deployment system | blue-green, rollback, approval |

> 💪 **Challenge**: Deploy Project 2 on a real server. Add Slack notifications. Then add a manual approval gate for production. That's a real DevOps pipeline that impresses interviewers!