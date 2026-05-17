---
title: "Kubernetes Projects — Beginner to Advanced (3 Real Projects)"
date: 2024-03-25
draft: false
description: "3 hands-on Kubernetes projects: beginner deploy your first app, intermediate microservices with Ingress and HPA, and advanced production GitOps with Helm and monitoring."
categories: ["kubernetes"]
tags: ["kubernetes", "k8s", "kubectl", "helm", "projects", "devops", "gitops"]
showToc: true
TocOpen: true
---

## Why These Projects? 🎯

Kubernetes is complex — the best way to learn it is by breaking things and fixing them. These 3 projects go from deploying your first pod to running a full production-grade microservices system with GitOps.

```
PROJECT ROADMAP:
──────────────────────────────────────────────────────────────
Project 1 (Beginner)     → Deploy Your First App on Kubernetes
Project 2 (Intermediate) → Microservices with Ingress + HPA
Project 3 (Advanced)     → Production GitOps with Helm + Monitoring
──────────────────────────────────────────────────────────────
```

---

## Project 1: Deploy Your First App on Kubernetes 🟢 Beginner

### What You'll Build

Deploy a Node.js API on a local Kubernetes cluster (minikube) — namespace, ConfigMap, Secret, Deployment, Service, and verify self-healing and scaling work correctly.

```
WHAT YOU'LL DEPLOY:
──────────────────────────────────────────────────────────────

Namespace: demo
    │
    ├── ConfigMap: app-config (LOG_LEVEL, APP_NAME)
    ├── Secret: app-secret (API_KEY, DB_PASSWORD)
    │
    └── Deployment: demo-app (3 replicas)
            │
            └── Service: demo-app-svc (NodePort)
                    │
                    └── http://minikube-ip:30080
──────────────────────────────────────────────────────────────
```

### Skills You'll Learn

- minikube setup
- Writing K8s YAML manifests
- kubectl commands
- Self-healing in action
- Scaling deployments
- ConfigMaps and Secrets

### Step 1: Setup minikube

```bash
# Install minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start cluster
minikube start --driver=docker --cpus=2 --memory=2048

# Verify
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   1m    v1.28.x

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard

# Open dashboard (optional)
minikube dashboard
```

### Step 2: Create the Application

```javascript
// src/app.js
const express = require('express');
const os = require('os');
const app = express();
app.use(express.json());

// Config from environment (injected by K8s)
const config = {
  appName: process.env.APP_NAME || 'K8s Demo',
  logLevel: process.env.LOG_LEVEL || 'info',
  environment: process.env.NODE_ENV || 'development',
  apiKey: process.env.API_KEY ? '***hidden***' : 'not set'
};

app.get('/', (req, res) => {
  res.json({
    message: `Hello from ${config.appName}!`,
    pod: os.hostname(),         // shows which pod handled request!
    environment: config.environment,
    logLevel: config.logLevel,
    version: '1.0.0'
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', pod: os.hostname() });
});

app.get('/ready', (req, res) => {
  res.json({ status: 'ready', pod: os.hostname() });
});

app.get('/config', (req, res) => {
  res.json(config);    // shows injected config
});

app.listen(3000, () => console.log(`🚀 ${config.appName} running on port 3000`));
```

```dockerfile
# Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src/ ./src/
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=5s \
  CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "src/app.js"]
```

```bash
# Build and push image
docker build -t pramendraatwork/k8s-demo:v1 .
docker push pramendraatwork/k8s-demo:v1

# Or use minikube's Docker daemon (no push needed!)
eval $(minikube docker-env)
docker build -t k8s-demo:v1 .
```

### Step 3: Create K8s Manifests

```bash
# Create project folder for manifests
mkdir k8s-first-app && cd k8s-first-app
```

```yaml
# 01-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
  labels:
    env: demo
    project: first-app
```

```yaml
# 02-configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: demo
data:
  APP_NAME: "K8s Demo App"
  LOG_LEVEL: "info"
  NODE_ENV: "production"
  MAX_CONNECTIONS: "100"
```

```yaml
# 03-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: demo
type: Opaque
stringData:              # stringData auto base64-encodes
  API_KEY: "my-super-secret-api-key"
  DB_PASSWORD: "supersecretdbpassword"
```

```yaml
# 04-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
  namespace: demo
  labels:
    app: demo-app
    version: v1
spec:
  replicas: 3

  selector:
    matchLabels:
      app: demo-app

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0        # zero downtime!

  template:
    metadata:
      labels:
        app: demo-app
        version: v1
    spec:
      containers:
        - name: demo-app
          image: pramendraatwork/k8s-demo:v1
          ports:
            - containerPort: 3000

          # Inject ConfigMap as env vars
          envFrom:
            - configMapRef:
                name: app-config

          # Inject specific Secrets
          env:
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: API_KEY
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD

          # Resource limits — always set these!
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"

          # Liveness probe — is the container alive?
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 10
            failureThreshold: 3

          # Readiness probe — ready to receive traffic?
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
```

```yaml
# 05-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-app-svc
  namespace: demo
  labels:
    app: demo-app
spec:
  selector:
    app: demo-app          # routes to pods with this label

  type: NodePort           # accessible from outside minikube

  ports:
    - protocol: TCP
      port: 80             # service port
      targetPort: 3000     # container port
      nodePort: 30080      # external port (30000-32767)
```

### Step 4: Deploy Everything

```bash
# Apply manifests in order
kubectl apply -f 01-namespace.yaml
kubectl apply -f 02-configmap.yaml
kubectl apply -f 03-secret.yaml
kubectl apply -f 04-deployment.yaml
kubectl apply -f 05-service.yaml

# Or apply entire directory at once
kubectl apply -f .

# Watch pods come up
kubectl get pods -n demo -w
# NAME                        READY   STATUS    RESTARTS   AGE
# demo-app-7d9f8b-abc12   0/1     Pending     0          1s
# demo-app-7d9f8b-abc12   0/1     Running     0          3s
# demo-app-7d9f8b-abc12   1/1     Running     0          8s

# Check all resources
kubectl get all -n demo
```

### Step 5: Access and Test the App

```bash
# Get minikube IP
minikube ip
# Example: 192.168.49.2

# Access the app
curl http://$(minikube ip):30080/
# {"message":"Hello from K8s Demo App!","pod":"demo-app-7d9f8b-abc12",...}

# Call multiple times — notice different pod names!
for i in {1..6}; do
  curl -s http://$(minikube ip):30080/ | python3 -c \
    "import sys,json; d=json.load(sys.stdin); print(f'Pod: {d[\"pod\"]}')"
done
# Pod: demo-app-7d9f8b-abc12
# Pod: demo-app-7d9f8b-def34   ← different pod!
# Pod: demo-app-7d9f8b-ghi56   ← different pod!

# Check config was injected
curl http://$(minikube ip):30080/config

# Or use minikube service command
minikube service demo-app-svc -n demo
```

### Step 6: Test Self-Healing

```bash
# List pods
kubectl get pods -n demo

# Delete a pod — watch K8s recreate it!
kubectl delete pod demo-app-7d9f8b-abc12 -n demo

# Watch immediately
kubectl get pods -n demo -w
# The deleted pod disappears, a NEW one appears automatically! ✅

# Kill the process inside a pod
kubectl exec -n demo demo-app-7d9f8b-abc12 -- kill 1

# K8s restarts the container automatically!
kubectl get pods -n demo
# RESTARTS column increments — proof it restarted!
```

### Step 7: Scale the Deployment

```bash
# Scale up to 6 replicas
kubectl scale deployment demo-app --replicas=6 -n demo
kubectl get pods -n demo -w

# Scale down to 1
kubectl scale deployment demo-app --replicas=1 -n demo

# Edit live (opens in vim)
kubectl edit deployment demo-app -n demo
# Change replicas: 3

# Watch the rolling update
kubectl rollout status deployment/demo-app -n demo
```

### Step 8: Rolling Update

```bash
# Build new version
docker build -t pramendraatwork/k8s-demo:v2 .
docker push pramendraatwork/k8s-demo:v2

# Update image (triggers rolling update)
kubectl set image deployment/demo-app \
  demo-app=pramendraatwork/k8s-demo:v2 \
  -n demo

# Watch rolling update (zero downtime!)
kubectl rollout status deployment/demo-app -n demo

# Check rollout history
kubectl rollout history deployment/demo-app -n demo

# Rollback if something went wrong
kubectl rollout undo deployment/demo-app -n demo

# Rollback to specific version
kubectl rollout undo deployment/demo-app --to-revision=1 -n demo
```

### Step 9: Debug Commands

```bash
# View logs
kubectl logs -n demo demo-app-7d9f8b-abc12
kubectl logs -n demo -l app=demo-app    # all pods with label
kubectl logs -n demo -l app=demo-app -f # follow all pods

# Describe pod (great for debugging!)
kubectl describe pod -n demo demo-app-7d9f8b-abc12
# Look at Events section at bottom!

# Shell into pod
kubectl exec -it -n demo demo-app-7d9f8b-abc12 -- sh

# Port forward (access pod directly)
kubectl port-forward -n demo deployment/demo-app 8080:3000
curl http://localhost:8080/

# Check resource usage
kubectl top pods -n demo
kubectl top nodes
```

### What You Learned

- ✅ minikube local cluster setup
- ✅ Namespace, ConfigMap, Secret
- ✅ Deployment with health checks
- ✅ NodePort Service
- ✅ Self-healing in action
- ✅ Scaling deployments
- ✅ Rolling updates and rollbacks

---

## Project 2: Microservices with Ingress + HPA 🟡 Intermediate

### What You'll Build

A microservices system — Frontend + Backend API + Database — with Ingress routing, Horizontal Pod Autoscaler, and persistent storage.

```
ARCHITECTURE:
──────────────────────────────────────────────────────────────

Browser
  │
  ▼
Ingress Controller (nginx)
  │
  ├── /api/*  ──▶  Backend Service  ──▶  Backend Pods (2-10)
  │                                              │
  │                                         PostgreSQL
  └── /*      ──▶  Frontend Service ──▶  Frontend Pods (2-5)

HPA watches CPU/Memory:
  Backend: scale 2→10 when CPU > 70%
  Frontend: scale 2→5 when CPU > 60%
──────────────────────────────────────────────────────────────
```

### Project Structure

```
microservices-k8s/
├── backend/
│   ├── src/app.js
│   └── Dockerfile
├── frontend/
│   ├── src/App.jsx
│   └── Dockerfile
└── k8s/
    ├── namespace.yaml
    ├── postgres/
    │   ├── secret.yaml
    │   ├── pvc.yaml
    │   ├── statefulset.yaml
    │   └── service.yaml
    ├── backend/
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── hpa.yaml
    ├── frontend/
    │   ├── deployment.yaml
    │   ├── service.yaml
    │   └── hpa.yaml
    └── ingress.yaml
```

### Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: microservices
  labels:
    project: microservices-demo
```

### PostgreSQL — StatefulSet with Persistent Storage

```yaml
# k8s/postgres/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: microservices
type: Opaque
stringData:
  POSTGRES_DB: appdb
  POSTGRES_USER: admin
  POSTGRES_PASSWORD: supersecretpassword
```

```yaml
# k8s/postgres/pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: microservices
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard   # minikube default
```

```yaml
# k8s/postgres/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: microservices
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16-alpine
          ports:
            - containerPort: 5432
          envFrom:
            - secretRef:
                name: postgres-secret
          env:
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          volumeMounts:
            - name: postgres-data
              mountPath: /var/lib/postgresql/data
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            exec:
              command:
                - pg_isready
                - -U
                - admin
                - -d
                - appdb
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command:
                - pg_isready
                - -U
                - admin
                - -d
                - appdb
            initialDelaySeconds: 5
            periodSeconds: 5
      volumes:
        - name: postgres-data
          persistentVolumeClaim:
            claimName: postgres-pvc
```

```yaml
# k8s/postgres/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: microservices
spec:
  selector:
    app: postgres
  clusterIP: None    # headless service for StatefulSet
  ports:
    - port: 5432
      targetPort: 5432
```

### Backend — Deployment + HPA

```yaml
# k8s/backend/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
  namespace: microservices
data:
  NODE_ENV: "production"
  PORT: "3000"
  DB_HOST: "postgres"
  DB_PORT: "5432"
  DB_NAME: "appdb"
  LOG_LEVEL: "info"
```

```yaml
# k8s/backend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: microservices
  labels:
    app: backend
spec:
  replicas: 2

  selector:
    matchLabels:
      app: backend

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: pramendraatwork/microservices-backend:v1
          ports:
            - containerPort: 3000

          envFrom:
            - configMapRef:
                name: backend-config

          env:
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: POSTGRES_PASSWORD

          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"

          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 20
            periodSeconds: 10

          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
```

```yaml
# k8s/backend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: microservices
spec:
  selector:
    app: backend
  type: ClusterIP    # internal only — Ingress handles external
  ports:
    - port: 80
      targetPort: 3000
```

```yaml
# k8s/backend/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: microservices
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend

  minReplicas: 2
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120
```

### Frontend — Deployment + HPA

```yaml
# k8s/frontend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: microservices
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: pramendraatwork/microservices-frontend:v1
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
```

```yaml
# k8s/frontend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
  namespace: microservices
spec:
  selector:
    app: frontend
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
```

```yaml
# k8s/frontend/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
  namespace: microservices
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

### Ingress

```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  namespace: microservices
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
spec:
  ingressClassName: nginx
  rules:
    - host: microservices.local    # add to /etc/hosts!
      http:
        paths:
          - path: /api(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: backend-svc
                port:
                  number: 80

          - path: /(.*)
            pathType: Prefix
            backend:
              service:
                name: frontend-svc
                port:
                  number: 80
```

### Deploy and Test

```bash
# Apply all manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/postgres/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/ingress.yaml

# Watch everything come up
kubectl get all -n microservices -w

# Add to /etc/hosts
echo "$(minikube ip) microservices.local" | sudo tee -a /etc/hosts

# Test endpoints
curl http://microservices.local/api/health
curl http://microservices.local/
curl http://microservices.local/api/tasks

# Check HPA
kubectl get hpa -n microservices

# Load test to trigger HPA scaling!
kubectl run load-test \
  --image=busybox \
  --rm -it \
  --restart=Never \
  -n microservices \
  -- sh -c "while true; do \
    wget -q -O- http://backend-svc/health; \
  done"

# In another terminal — watch pods scale up!
kubectl get pods -n microservices -w
kubectl get hpa -n microservices -w

# Check PVC
kubectl get pvc -n microservices
kubectl describe pvc postgres-pvc -n microservices

# Verify postgres data persists
kubectl exec -it -n microservices postgres-0 -- \
  psql -U admin -d appdb -c "SELECT COUNT(*) FROM tasks;"

# Delete postgres pod — StatefulSet recreates with same data!
kubectl delete pod postgres-0 -n microservices
kubectl get pods -n microservices -w
# postgres-0 recreates and mounts same PVC — data safe!
```

### What You Learned

- ✅ StatefulSet for databases
- ✅ PersistentVolumeClaim for storage
- ✅ Ingress with path-based routing
- ✅ HPA auto-scaling with behavior rules
- ✅ Multi-service architecture
- ✅ ClusterIP vs NodePort vs Ingress
- ✅ Load testing to trigger autoscaling

---

## Project 3: Production GitOps with Helm + Monitoring 🔴 Advanced

### What You'll Build

A complete production Kubernetes setup — Helm charts for deployment, Prometheus + Grafana monitoring, RBAC security, resource quotas, and a full GitOps CI/CD pipeline.

```
PRODUCTION ARCHITECTURE:
──────────────────────────────────────────────────────────────

GitHub Push
    │
    ▼
GitHub Actions CI
    │ Build + Test + Push image
    ▼
Update Helm values.yaml
    │ (auto commit)
    ▼
ArgoCD detects change
    │ (or kubectl apply in pipeline)
    ▼
┌─────────────────────────────────────────────┐
│           KUBERNETES CLUSTER                 │
│                                             │
│  Namespace: production                      │
│  ┌───────────────────────────────────────┐  │
│  │  Helm Release: my-app                 │  │
│  │  ┌──────────┐  ┌──────────────────┐  │  │
│  │  │  App     │  │   PostgreSQL      │  │  │
│  │  │  (5 pods)│  │   (StatefulSet)   │  │  │
│  │  └──────────┘  └──────────────────┘  │  │
│  └───────────────────────────────────────┘  │
│                                             │
│  Namespace: monitoring                      │
│  ┌───────────────────────────────────────┐  │
│  │  Prometheus + Grafana + AlertManager  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
──────────────────────────────────────────────────────────────
```

### Step 1: Install Helm

```bash
# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version

# Add popular repos
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Step 2: Create Your Own Helm Chart

```bash
# Scaffold chart
helm create my-app

# Structure created:
my-app/
├── Chart.yaml          # chart metadata
├── values.yaml         # default values
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   ├── configmap.yaml
│   ├── serviceaccount.yaml
│   └── _helpers.tpl    # reusable template functions
└── charts/             # sub-chart dependencies
```

```yaml
# my-app/Chart.yaml
apiVersion: v2
name: my-app
description: Production-ready Helm chart for my-app
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - nodejs
  - api
  - devops
maintainers:
  - name: Pramendra Rajput
    email: pramendraatwork@gmail.com
dependencies:
  - name: postgresql
    version: "13.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
```

```yaml
# my-app/values.yaml
# Default values — override per environment

replicaCount: 3

image:
  repository: pramendraatwork/my-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  name: ""

service:
  type: ClusterIP
  port: 80
  targetPort: 3000

ingress:
  enabled: true
  className: nginx
  host: myapp.example.com
  tls:
    enabled: false
    secretName: myapp-tls

resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

config:
  nodeEnv: production
  logLevel: info
  port: "3000"

secrets:
  dbPassword: ""     # override with --set or secret values file
  apiKey: ""

postgresql:
  enabled: true
  auth:
    database: appdb
    username: admin
    password: ""     # set via secrets
  primary:
    persistence:
      size: 10Gi

nodeSelector: {}
tolerations: []
affinity:
  podAntiAffinity:          # spread pods across nodes!
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app.kubernetes.io/name
                operator: In
                values:
                  - my-app
          topologyKey: kubernetes.io/hostname

livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 20
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 3
```

```yaml
# my-app/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    deployment.kubernetes.io/revision: "{{ .Release.Revision }}"
    app.kubernetes.io/version: {{ .Values.image.tag | quote }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      serviceAccountName: {{ include "my-app.serviceAccountName" . }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.config.port | int }}
          env:
            - name: NODE_ENV
              value: {{ .Values.config.nodeEnv | quote }}
            - name: LOG_LEVEL
              value: {{ .Values.config.logLevel | quote }}
            - name: PORT
              value: {{ .Values.config.port | quote }}
            {{- if .Values.secrets.dbPassword }}
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "my-app.fullname" . }}-secrets
                  key: DB_PASSWORD
            {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          livenessProbe:
            {{- toYaml .Values.livenessProbe | nindent 12 }}
          readinessProbe:
            {{- toYaml .Values.readinessProbe | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

```yaml
# my-app/templates/hpa.yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "my-app.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "my-app.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
{{- end }}
```

### Step 3: Environment-Specific Values

```yaml
# values-staging.yaml
replicaCount: 2
image:
  tag: "staging"
ingress:
  host: staging.myapp.com
resources:
  requests:
    memory: "64Mi"
    cpu: "50m"
  limits:
    memory: "128Mi"
    cpu: "200m"
autoscaling:
  minReplicas: 1
  maxReplicas: 5
```

```yaml
# values-production.yaml
replicaCount: 5
image:
  tag: "1.0.0"
ingress:
  host: myapp.com
  tls:
    enabled: true
    secretName: myapp-tls
resources:
  requests:
    memory: "256Mi"
    cpu: "200m"
  limits:
    memory: "512Mi"
    cpu: "1000m"
autoscaling:
  minReplicas: 3
  maxReplicas: 20
```

### Step 4: RBAC Setup

```yaml
# rbac.yaml — control who can do what
---
# ServiceAccount for CI/CD pipeline
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cicd-deployer
  namespace: production

---
# Role — what the pipeline can do
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployer-role
  namespace: production
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "update", "patch"]
  - apiGroups: [""]
    resources: ["services", "configmaps"]
    verbs: ["get", "list"]
  - apiGroups: ["autoscaling"]
    resources: ["horizontalpodautoscalers"]
    verbs: ["get", "list"]

---
# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployer-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: cicd-deployer
    namespace: production
roleRef:
  kind: Role
  name: deployer-role
  apiGroup: rbac.authorization.k8s.io

---
# ResourceQuota — prevent runaway resource usage
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    pods: "50"
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    count/deployments.apps: "20"
```

### Step 5: Install Monitoring Stack

```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Install Prometheus + Grafana + AlertManager (kube-prometheus-stack)
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.adminPassword=admin123 \
  --set prometheus.prometheusSpec.retention=7d \
  --set alertmanager.alertmanagerSpec.retention=24h

# Watch it come up
kubectl get pods -n monitoring -w

# Access Grafana
kubectl port-forward -n monitoring \
  service/monitoring-grafana 3000:80
# Open: http://localhost:3000
# Login: admin / admin123

# Access Prometheus
kubectl port-forward -n monitoring \
  service/monitoring-kube-prometheus-prometheus 9090:9090
# Open: http://localhost:9090
```

### Step 6: Deploy with Helm

```bash
# Validate chart
helm lint my-app/

# Dry run — see what would be deployed
helm install my-app ./my-app \
  --namespace production \
  --create-namespace \
  --dry-run \
  --debug

# Install for real
helm install my-app ./my-app \
  --namespace production \
  --create-namespace \
  --values my-app/values-production.yaml \
  --set secrets.dbPassword=supersecret \
  --set image.tag=1.0.0

# Check release
helm list -n production
helm status my-app -n production

# Upgrade (new image version)
helm upgrade my-app ./my-app \
  --namespace production \
  --values my-app/values-production.yaml \
  --set image.tag=1.1.0

# Watch rollout
kubectl rollout status deployment/my-app -n production

# Rollback with Helm
helm rollback my-app 1 -n production   # rollback to revision 1
helm history my-app -n production       # see all revisions

# Uninstall
helm uninstall my-app -n production
```

### Step 7: GitOps CI/CD Pipeline

```yaml
# .github/workflows/deploy-k8s.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}

    steps:
      - uses: actions/checkout@v4

      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - uses: docker/metadata-action@v5
        id: meta
        with:
          images: pramendraatwork/my-app
          tags: |
            type=sha,prefix=
            type=semver,pattern={{version}}

      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: Install Helm
        uses: azure/setup-helm@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
          echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV

      - name: Deploy to staging
        run: |
          helm upgrade --install my-app ./my-app \
            --namespace staging \
            --create-namespace \
            --values my-app/values-staging.yaml \
            --set image.tag=${{ needs.build-and-push.outputs.image-tag }} \
            --set secrets.dbPassword=${{ secrets.DB_PASSWORD }} \
            --wait \
            --timeout 5m

      - name: Verify deployment
        run: |
          kubectl rollout status deployment/my-app \
            -n staging --timeout=3m
          kubectl get pods -n staging

  deploy-production:
    needs: [build-and-push, deploy-staging]
    runs-on: ubuntu-latest
    environment: production    # requires manual approval!

    steps:
      - uses: actions/checkout@v4

      - name: Install Helm
        uses: azure/setup-helm@v3

      - name: Configure kubectl
        run: |
          echo "${{ secrets.PROD_KUBECONFIG }}" | base64 -d > kubeconfig
          echo "KUBECONFIG=$(pwd)/kubeconfig" >> $GITHUB_ENV

      - name: Deploy to production
        run: |
          helm upgrade --install my-app ./my-app \
            --namespace production \
            --create-namespace \
            --values my-app/values-production.yaml \
            --set image.tag=${{ needs.build-and-push.outputs.image-tag }} \
            --set secrets.dbPassword=${{ secrets.DB_PASSWORD }} \
            --atomic \
            --timeout 10m
          # --atomic: rollback automatically on failure!

      - name: Production health check
        run: |
          kubectl wait pod \
            -l app.kubernetes.io/name=my-app \
            -n production \
            --for=condition=Ready \
            --timeout=120s

          echo "✅ All pods healthy!"
          kubectl get pods -n production

      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: ${{ needs.build-and-push.outputs.image-tag }}
          generate_release_notes: true
```

### Useful Helm Commands Reference

```bash
# Search charts
helm search repo nginx
helm search hub postgres

# Show chart info
helm show chart bitnami/postgresql
helm show values bitnami/postgresql

# Get values of installed release
helm get values my-app -n production

# Test chart (runs test pods)
helm test my-app -n production

# Template rendering (debug)
helm template my-app ./my-app \
  --values values-production.yaml \
  --debug

# Package chart for distribution
helm package ./my-app

# Push to chart registry (OCI)
helm push my-app-1.0.0.tgz oci://registry-1.docker.io/pramendraatwork
```

### What You Learned

- ✅ Helm chart creation from scratch
- ✅ Environment-specific values files
- ✅ Helm template functions and conditionals
- ✅ Helm dependencies (postgresql sub-chart)
- ✅ RBAC — least-privilege access
- ✅ ResourceQuota for namespaces
- ✅ kube-prometheus-stack monitoring
- ✅ GitOps CI/CD with Helm
- ✅ Automatic rollback with `--atomic`
- ✅ Production pod affinity rules

---

## Summary 📋

| Project | Level | What You Built | Key Skills |
|---|---|---|---|
| First App | 🟢 Beginner | App on minikube with self-healing | namespace, deploy, service, scale |
| Microservices | 🟡 Intermediate | Multi-service + Ingress + HPA | StatefulSet, PVC, autoscaling |
| GitOps + Helm | 🔴 Advanced | Full prod with monitoring + CI/CD | Helm, RBAC, Prometheus, GitOps |

> 💪 **Challenge**: Complete all 3 projects on minikube. Then take Project 3 and deploy it on AWS EKS free tier. Add Grafana dashboards showing your pod CPU and memory. That's a real production Kubernetes portfolio that stands out in any DevOps interview!