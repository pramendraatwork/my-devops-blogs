---
title: "Jenkins — Complete CI/CD Guide from Beginner to Advanced"
date: 2024-02-20
draft: false
description: "Everything about Jenkins: what it is, installation, pipelines, Jenkinsfile, plugins, and real DevOps workflows with diagrams."
categories: ["jenkins"]
tags: ["jenkins", "ci-cd", "pipeline", "automation", "devops"]
showToc: true
TocOpen: true
---

## 1. What is Jenkins? 🤖

Jenkins is the world's most popular **open-source automation server**. It automates everything in your software delivery process — building, testing, and deploying code.

> 💡 **Simple analogy**: Jenkins is like a **robot employee** that never sleeps. Every time you push code, Jenkins wakes up, builds your app, runs tests, and deploys it — automatically.

### Without Jenkins vs With Jenkins

```
WITHOUT JENKINS (manual hell 😩)
─────────────────────────────────────────────
Developer pushes code
    │
    ▼
Someone manually runs tests (maybe forgets)
    │
    ▼
Someone manually builds the app
    │
    ▼
Someone manually deploys to server (at 2am 😭)
    │
    ▼
Something breaks in production
    │
    ▼
Everyone panics 🔥

WITH JENKINS (automated bliss 😊)
─────────────────────────────────────────────
Developer pushes code
    │
    ▼
Jenkins automatically detects the push
    │
    ▼
Jenkins runs tests (fails? notifies developer!)
    │
    ▼
Jenkins builds the app
    │
    ▼
Jenkins deploys to staging/production
    │
    ▼
Team gets notified ✅
    │
    ▼
Everyone sleeps peacefully 😴
```

### Jenkins vs GitHub Actions vs GitLab CI

| Feature | Jenkins | GitHub Actions | GitLab CI |
|---|---|---|---|
| **Type** | Self-hosted | Cloud (GitHub) | Cloud/Self-hosted |
| **Cost** | Free (infra cost) | Free tier + paid | Free tier + paid |
| **Setup** | Manual | Zero setup | Zero setup |
| **Plugins** | 1800+ plugins | Marketplace | Built-in |
| **Flexibility** | Maximum | High | High |
| **Learning curve** | Steep | Moderate | Moderate |
| **Best for** | Enterprise, complex | GitHub projects | GitLab projects |

---

## 2. Jenkins Architecture 🏗️

```
┌─────────────────────────────────────────────────────────┐
│                  JENKINS ARCHITECTURE                    │
│                                                         │
│   ┌─────────────────────────────────┐                  │
│   │        Jenkins Master            │                  │
│   │  • Schedules builds              │                  │
│   │  • Manages agents                │                  │
│   │  • Web UI (port 8080)            │                  │
│   │  • Stores config & history       │                  │
│   └──────────┬──────────────────────┘                  │
│              │                                          │
│    ┌─────────┼──────────┐                              │
│    ▼         ▼          ▼                              │
│ ┌──────┐ ┌──────┐ ┌──────┐                            │
│ │Agent │ │Agent │ │Agent │                            │
│ │  1   │ │  2   │ │  3   │                            │
│ │Linux │ │Win   │ │Mac   │                            │
│ └──────┘ └──────┘ └──────┘                            │
│  Runs      Runs      Runs                              │
│  builds    builds    builds                            │
└─────────────────────────────────────────────────────────┘

Master = Brain (coordinates everything)
Agent  = Worker (actually runs the builds)
```

### Key Concepts

| Term | What it means |
|---|---|
| **Job/Project** | A task Jenkins runs (build, test, deploy) |
| **Build** | One execution of a job |
| **Pipeline** | Series of stages (build → test → deploy) |
| **Stage** | A phase in the pipeline |
| **Step** | A single command within a stage |
| **Agent** | Machine that runs the build |
| **Workspace** | Directory where build runs |
| **Artifact** | Output of a build (JAR, Docker image) |
| **Plugin** | Extension that adds functionality |

---

## 3. Installing Jenkins 🔧

### On Ubuntu/Debian (Recommended)

```bash
# Step 1: Install Java (Jenkins requires Java)
sudo apt update
sudo apt install -y fontconfig openjdk-17-jre
java -version   # verify: openjdk 17...

# Step 2: Add Jenkins repository
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# Step 3: Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Step 4: Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins    # start on boot
sudo systemctl status jenkins    # verify running

# Step 5: Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### On Docker (Fastest Way)

```bash
# Run Jenkins in Docker
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

# Get initial password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Initial Setup

```
1. Open browser: http://localhost:8080
2. Enter initial admin password
3. Install suggested plugins (recommended for beginners)
4. Create first admin user
5. Set Jenkins URL
6. Start using Jenkins! 🎉
```

---

## 4. Jenkins Pipeline — The Heart of Jenkins 💓

A **Pipeline** is the modern way to define your CI/CD process as code in a file called `Jenkinsfile`.

### Pipeline Types

```
DECLARATIVE (recommended — cleaner syntax)
──────────────────────────────────────────
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
    }
}

SCRIPTED (older — more flexible but complex)
──────────────────────────────────────────
node {
    stage('Build') {
        echo 'Building...'
    }
}
```

### Basic Jenkinsfile

```groovy
// Jenkinsfile (save in root of your repo)
pipeline {
    agent any    // run on any available agent

    environment {
        APP_NAME = 'my-app'
        VERSION  = '1.0.0'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Checking out code...'
                checkout scm    // checkout from configured SCM
            }
        }

        stage('Build') {
            steps {
                echo '🔨 Building application...'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                sh 'npm test'
            }
            post {
                always {
                    junit 'test-results/*.xml'    // publish test results
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deploying application...'
                sh './deploy.sh'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
        always {
            echo '🧹 Cleaning up...'
            cleanWs()    // clean workspace
        }
    }
}
```

---

## 5. Real World Jenkinsfile 🌍

### Node.js App — Full Pipeline

```groovy
pipeline {
    agent any

    tools {
        nodejs 'NodeJS-20'    // configured in Jenkins tools
    }

    environment {
        DOCKER_IMAGE    = 'myrepo/my-app'
        DOCKER_TAG      = "${env.BUILD_NUMBER}"
        REGISTRY_CREDS  = credentials('dockerhub-credentials')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    stages {
        // ── Stage 1: Checkout ──────────────────────────────
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/user/my-app.git',
                    credentialsId: 'github-credentials'
            }
        }

        // ── Stage 2: Install Dependencies ─────────────────
        stage('Install') {
            steps {
                sh 'npm ci'    // faster than npm install
            }
        }

        // ── Stage 3: Lint ──────────────────────────────────
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        // ── Stage 4: Test ──────────────────────────────────
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
            }
        }

        // ── Stage 5: Build Docker Image ────────────────────
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        // ── Stage 6: Push to Registry ─────────────────────
        stage('Push Image') {
            when {
                branch 'main'    // only push on main branch
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        // ── Stage 7: Deploy to Staging ─────────────────────
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            steps {
                sshagent(['staging-server-key']) {
                    sh """
                        ssh ubuntu@staging.example.com '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop app || true
                            docker rm app || true
                            docker run -d --name app \
                                -p 3000:3000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }

        // ── Stage 8: Deploy to Production ─────────────────
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            input {
                message "Deploy to production?"
                ok "Yes, deploy!"
                submitter "admin,devops-team"
            }
            steps {
                sshagent(['prod-server-key']) {
                    sh """
                        ssh ubuntu@prod.example.com '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop app || true
                            docker rm app || true
                            docker run -d --name app \
                                -p 3000:3000 \
                                --restart unless-stopped \
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ Build #${env.BUILD_NUMBER} succeeded! ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ Build #${env.BUILD_NUMBER} failed! ${env.BUILD_URL}"
            )
            emailext(
                subject: "Build Failed: ${env.JOB_NAME}",
                body: "Build ${env.BUILD_NUMBER} failed. Check: ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
        always {
            cleanWs()
        }
    }
}
```

---

## 6. Pipeline Syntax Deep Dive 📚

### Agent

```groovy
// Run on any agent
agent any

// Run on specific label
agent { label 'linux' }
agent { label 'docker && linux' }

// Run in Docker container
agent {
    docker {
        image 'node:20-alpine'
        args '-v /tmp:/tmp'
    }
}

// No agent (define per stage)
agent none
```

### When — Conditional Execution

```groovy
stage('Deploy') {
    when {
        branch 'main'                          // only on main branch
    }
}

stage('Release') {
    when {
        tag "v*"                               // only on version tags
    }
}

stage('Notify') {
    when {
        anyOf {
            branch 'main'
            branch 'develop'
        }
    }
}

stage('Skip on Weekend') {
    when {
        not {
            triggeredBy 'TimerTrigger'
        }
    }
}
```

### Parallel Stages

```groovy
stage('Tests') {
    parallel {
        stage('Unit') {
            steps { sh 'npm run test:unit' }
        }
        stage('E2E') {
            steps { sh 'npm run test:e2e' }
        }
        stage('Security Scan') {
            steps { sh 'npm audit' }
        }
    }
}
```

### Parameters — User Input

```groovy
parameters {
    string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
    choice(name: 'ENVIRONMENT', choices: ['staging', 'production'], description: 'Target env')
    booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
}

// Use parameters
sh "docker pull myapp:${params.VERSION}"
```

### Credentials — Secrets Management

```groovy
environment {
    // Username + password
    DB_CREDS = credentials('database-credentials')
    // DB_CREDS_USR = username
    // DB_CREDS_PSW = password

    // Secret text
    API_KEY = credentials('api-key-secret')

    // SSH key
    SSH_KEY = credentials('prod-ssh-key')
}

steps {
    withCredentials([
        usernamePassword(
            credentialsId: 'dockerhub',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )
    ]) {
        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
    }
}
```

---

## 7. Jenkins Plugins — Must Have 🔌

```
Essential Plugins:
──────────────────────────────────────────────────
Plugin                    | What it does
──────────────────────────────────────────────────
Git Plugin                | Connect to Git repos
GitHub Integration        | GitHub webhooks & PRs
Pipeline                  | Declarative pipelines
Blue Ocean                | Beautiful pipeline UI
Docker Pipeline           | Build/push Docker images
Credentials Binding       | Inject secrets safely
SSH Agent                 | SSH key management
Slack Notification        | Send Slack messages
Email Extension           | Send emails
NodeJS                    | Node.js tool management
Maven Integration         | Java/Maven builds
Kubernetes                | Run agents in K8s pods
AWS Steps                 | Interact with AWS
SonarQube Scanner         | Code quality analysis
OWASP Dependency Check    | Security scanning
──────────────────────────────────────────────────
```

---

## 8. Jenkins + GitHub Webhook 🔗

Set up automatic builds on every git push:

```
WEBHOOK FLOW:
─────────────────────────────────────────────
Developer pushes code to GitHub
    │
    ▼
GitHub sends webhook to Jenkins
(POST http://jenkins-url/github-webhook/)
    │
    ▼
Jenkins receives webhook
    │
    ▼
Jenkins triggers the pipeline
    │
    ▼
Build runs automatically ✅
```

### Setup Steps

```
1. In Jenkins:
   • Job → Configure → Build Triggers
   • ✅ GitHub hook trigger for GITScm polling

2. In GitHub repo:
   • Settings → Webhooks → Add webhook
   • Payload URL: http://YOUR_JENKINS_URL/github-webhook/
   • Content type: application/json
   • Events: Just the push event
   • ✅ Active → Add webhook

3. Test it:
   • Push a commit
   • Watch Jenkins automatically trigger! 🚀
```

---

## 9. Shared Libraries — Reusable Code 📦

Shared Libraries let you reuse pipeline code across multiple projects.

```
jenkins-shared-library/
├── vars/
│   ├── buildApp.groovy       ← global function
│   ├── deployApp.groovy
│   └── sendNotification.groovy
├── src/
│   └── com/company/
│       └── Utils.groovy      ← helper classes
└── resources/
    └── scripts/
        └── deploy.sh
```

```groovy
// vars/buildApp.groovy
def call(String appName, String version) {
    sh "docker build -t ${appName}:${version} ."
    sh "docker push ${appName}:${version}"
}

// Using it in Jenkinsfile (any repo)
@Library('jenkins-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp('my-app', '1.0.0')
            }
        }
    }
}
```

---

## 10. Jenkins Best Practices ✅

```
DO ✅                              DON'T ❌
─────────────────────────────────────────────
Store Jenkinsfile in repo          Store pipeline in Jenkins UI
Use declarative pipelines          Use scripted pipelines (unless needed)
Use shared libraries               Copy-paste pipeline code
Store secrets in credentials       Hardcode passwords in Jenkinsfile
Set build timeouts                 Let builds run forever
Clean workspace after build        Leave build artifacts
Use parallel stages                Run everything sequentially
Pin plugin versions                Always use latest plugins
Use agents for builds              Run everything on master
Back up Jenkins regularly          Hope nothing breaks
```

---

## 11. Troubleshooting Jenkins 🔧

```bash
# Jenkins logs
sudo journalctl -u jenkins -f
sudo cat /var/log/jenkins/jenkins.log
tail -f /var/lib/jenkins/jobs/my-job/builds/1/log

# Common issues and fixes:

# ── Permission denied ─────────────────────────────────────
sudo usermod -aG docker jenkins     # add jenkins to docker group
sudo systemctl restart jenkins

# ── Out of disk space ────────────────────────────────────
# Clean old builds: Manage Jenkins → System → Build History
# Or in Jenkinsfile:
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
}

# ── Plugin conflicts ─────────────────────────────────────
# Manage Jenkins → Plugin Manager → check for updates
# Restart Jenkins after updates

# ── Agent offline ────────────────────────────────────────
# Manage Jenkins → Nodes → Click agent → Launch agent
# Check agent logs for connection issues

# ── Build stuck ──────────────────────────────────────────
# Set timeout in pipeline:
options {
    timeout(time: 30, unit: 'MINUTES')
}
```

---

## 12. Quick Reference 📋

```groovy
// BASIC PIPELINE STRUCTURE
pipeline {
    agent any
    environment { KEY = 'value' }
    options { timeout(time: 30, unit: 'MINUTES') }
    parameters { string(name: 'VER', defaultValue: '1.0') }
    triggers { pollSCM('H/5 * * * *') }

    stages {
        stage('Name') {
            when { branch 'main' }
            steps {
                sh 'command'
                echo 'message'
                script { /* groovy code */ }
            }
        }
    }

    post {
        always  { cleanWs() }
        success { echo 'done!' }
        failure { echo 'failed!' }
    }
}

// USEFUL BUILT-IN VARIABLES
env.BUILD_NUMBER      // build number
env.BUILD_URL         // URL to this build
env.JOB_NAME          // job name
env.BRANCH_NAME       // git branch
env.GIT_COMMIT        // git commit hash
env.WORKSPACE         // build directory path
```

---

## What's Next? 🚀

Jenkins is the foundation. Now layer these on top:

- **Docker** — Jenkins builds Docker images
- **Kubernetes** — Jenkins deploys to K8s clusters
- **SonarQube** — add code quality gates to pipeline
- **AWS** — Jenkins deploys to EC2, ECS, Lambda

> 💪 **Practice tip**: Install Jenkins locally with Docker, create a simple pipeline for any project, and add one stage at a time. The best way to learn Jenkins is to break it and fix it!