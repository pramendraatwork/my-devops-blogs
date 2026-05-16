---
title: "Kubernetes Part 1 — Beginner's Complete Guide to Container Orchestration"
date: 2024-03-10
draft: false
description: "Kubernetes from absolute zero: what it is, architecture, core objects, kubectl commands, namespaces, and your first real deployment project."
categories: ["kubernetes"]
tags: ["kubernetes", "k8s", "kubectl", "pods", "deployments", "beginner", "devops"]
showToc: true
TocOpen: true
---

## 1. What is Kubernetes? ☸️

You've learned Docker. You can run containers. But now imagine you have:
- 50 containers running your app
- One crashes at 3am — who restarts it?
- Traffic spikes — who adds more containers?
- A server dies — who moves containers to healthy servers?

**You need an orchestrator. That's Kubernetes.**

> 💡 **Simple definition**: Kubernetes (K8s) is a system that manages containers automatically — starting them, stopping them, scaling them, healing them, and distributing them across multiple machines.

### The Problem Kubernetes Solves

```
WITHOUT KUBERNETES:
──────────────────────────────────────────────────────────────

Server 1          Server 2          Server 3
────────          ────────          ────────
[Container 1] ✅  [Container 4] ✅  [Container 7] ✅
[Container 2] ✅  [Container 5] 💀  [Container 8] ✅
[Container 3] ✅  [Container 6] ✅  [Container 9] 💀

Container 5 crashed! Nobody knows.
Container 9 crashed! Server 3 is overloaded.
You get paged at 3am. 😭

WITH KUBERNETES:
──────────────────────────────────────────────────────────────

Kubernetes detects Container 5 crashed → restarts it automatically ✅
Kubernetes detects Server 3 overloaded → moves container to Server 1 ✅
Kubernetes detects traffic spike → adds 5 more containers ✅
You sleep peacefully. 😴
```

### What Kubernetes Does For You

```
┌─────────────────────────────────────────────────────────────┐
│                KUBERNETES SUPERPOWERS                       │
│                                                             │
│  🔄 Self-healing      → restarts crashed containers        │
│  📈 Auto-scaling      → adds/removes containers on demand  │
│  ⚖️  Load balancing   → distributes traffic evenly         │
│  🚀 Rolling updates   → zero-downtime deployments          │
│  ⏪ Rollbacks         → instantly revert bad deploys       │
│  🏥 Health checks     → only sends traffic to healthy pods │
│  🔒 Secret management → securely handles passwords/keys    │
│  💾 Storage mgmt      → manages persistent storage         │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Kubernetes vs Docker Compose 🆚

```
DOCKER COMPOSE                    KUBERNETES
──────────────────────────────────────────────────────────────
Single machine only               Multiple machines (cluster)
Manual scaling                    Auto-scaling
No self-healing                   Self-healing (restarts pods)
Simple YAML                       More complex YAML
Local development                 Production at scale
No built-in load balancing        Built-in load balancing
No rolling updates                Rolling updates built-in
Simple setup                      Complex setup
Good for: dev/test                Good for: production

RULE OF THUMB:
• 1-2 servers, simple app  → Docker Compose
• 3+ servers, production   → Kubernetes
```

---

## 3. Kubernetes Architecture 🏗️

```
KUBERNETES CLUSTER ARCHITECTURE
──────────────────────────────────────────────────────────────

                    ┌─────────────────────────────┐
                    │      CONTROL PLANE           │
                    │      (Master Node)           │
                    │                             │
                    │  ┌─────────┐ ┌───────────┐  │
                    │  │  API    │ │ Scheduler  │  │
                    │  │ Server  │ │           │  │
                    │  └─────────┘ └───────────┘  │
                    │  ┌─────────┐ ┌───────────┐  │
                    │  │  etcd   │ │Controller │  │
                    │  │  (DB)   │ │ Manager   │  │
                    │  └─────────┘ └───────────┘  │
                    └──────────────┬──────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
          ┌─────────▼──┐  ┌───────▼────┐  ┌─────▼──────┐
          │  Worker 1  │  │  Worker 2  │  │  Worker 3  │
          │            │  │            │  │            │
          │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │
          │ │  Pod   │ │  │ │  Pod   │ │  │ │  Pod   │ │
          │ │  Pod   │ │  │ │  Pod   │ │  │ │  Pod   │ │
          │ └────────┘ │  │ └────────┘ │  │ └────────┘ │
          │  kubelet   │  │  kubelet   │  │  kubelet   │
          │  kube-proxy│  │  kube-proxy│  │  kube-proxy│
          └────────────┘  └────────────┘  └────────────┘
```

### Control Plane Components

| Component | What it does |
|---|---|
| **API Server** | Front door of K8s. All communication goes through it. |
| **etcd** | Key-value database. Stores ALL cluster state. |
| **Scheduler** | Decides which worker node runs each pod. |
| **Controller Manager** | Watches cluster state, fixes differences. |

### Worker Node Components

| Component | What it does |
|---|---|
| **kubelet** | Agent on each node. Runs pods, reports to master. |
| **kube-proxy** | Handles networking, load balancing between pods. |
| **Container Runtime** | Actually runs containers (containerd, Docker). |

---

## 4. Core Kubernetes Objects 📦

### Pod — The Smallest Unit

```
POD:
──────────────────────────────────────────────────────────────

┌─────────────────────────────────┐
│              POD                │
│  ┌─────────────┐ ┌───────────┐ │
│  │  Container  │ │ Sidecar   │ │
│  │  (your app) │ │(log agent)│ │
│  └─────────────┘ └───────────┘ │
│                                 │
│  Shared network (same IP)       │
│  Shared storage (same volumes)  │
│  Same lifecycle                 │
└─────────────────────────────────┘

• Smallest deployable unit in K8s
• Usually 1 container per pod
• Gets its own IP address
• Ephemeral — can die anytime!
• Never create pods directly — use Deployments
```

```yaml
# pod.yaml — basic pod (don't use in production!)
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: my-container
      image: nginx:1.25
      ports:
        - containerPort: 80
```

### Deployment — Managing Pods

```
DEPLOYMENT:
──────────────────────────────────────────────────────────────

Deployment (desired: 3 replicas)
│
├── ReplicaSet (ensures 3 pods always running)
│   ├── Pod 1 ✅
│   ├── Pod 2 ✅
│   └── Pod 3 ✅
│
If Pod 2 dies:
├── ReplicaSet detects: only 2 pods!
└── Creates Pod 4 automatically ✅

Deployment also handles:
• Rolling updates (zero downtime)
• Rollbacks (go back to previous version)
• Scaling (increase/decrease replicas)
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: default
  labels:
    app: my-app
spec:
  replicas: 3                    # run 3 copies

  selector:
    matchLabels:
      app: my-app                # manage pods with this label

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1                # max extra pods during update
      maxUnavailable: 0          # zero downtime!

  template:                      # pod template
    metadata:
      labels:
        app: my-app              # must match selector!
    spec:
      containers:
        - name: my-app
          image: myrepo/my-app:1.0.0
          ports:
            - containerPort: 3000

          # Resource limits (always set these!)
          resources:
            requests:
              memory: "128Mi"    # minimum guaranteed
              cpu: "100m"        # 100 millicores = 0.1 CPU
            limits:
              memory: "256Mi"    # maximum allowed
              cpu: "500m"        # 500 millicores = 0.5 CPU

          # Health checks
          livenessProbe:         # is container alive?
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 10

          readinessProbe:        # is container ready for traffic?
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5

          # Environment variables
          env:
            - name: NODE_ENV
              value: "production"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
```

### Service — Stable Network Endpoint

```
WHY WE NEED SERVICES:
──────────────────────────────────────────────────────────────

PROBLEM:
Pod IPs change every time a pod restarts!
Pod 1: 10.0.0.1  → dies → new Pod: 10.0.0.5
How does your frontend know where to find the backend?

SOLUTION — Service:
Frontend → Service (stable IP: 10.96.0.1) → Pod 1 or Pod 2 or Pod 3
              │
              └── Load balances automatically!
                  Pod IP changes don't matter!
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app              # routes to pods with this label

  type: ClusterIP            # internal only (default)

  ports:
    - protocol: TCP
      port: 80               # service port
      targetPort: 3000       # container port
```

**Service Types:**

```
ClusterIP (default):
  • Internal only — only accessible within cluster
  • Use for: backend services, databases
  • Example: frontend → backend service

NodePort:
  • Exposes on each node's IP + a port (30000-32767)
  • Accessible from outside cluster
  • Use for: development, testing
  • URL: http://node-ip:30080

LoadBalancer:
  • Creates cloud load balancer (AWS ELB, GCP LB)
  • Gets public IP automatically
  • Use for: production web services
  • Most expensive option

ExternalName:
  • Maps service to external DNS name
  • Use for: accessing external services
```

### ConfigMap — Non-Secret Configuration

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Key-value pairs
  LOG_LEVEL: "info"
  API_URL: "https://api.example.com"
  MAX_CONNECTIONS: "100"

  # File content
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://backend;
      }
    }
```

```yaml
# Use ConfigMap in pod
spec:
  containers:
    - name: app
      # As environment variables
      envFrom:
        - configMapRef:
            name: app-config

      # Or specific keys
      env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL

      # Or as mounted file
      volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf

  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

### Secret — Sensitive Data

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Values must be base64 encoded!
  # echo -n 'mypassword' | base64
  username: bXl1c2Vy          # myuser
  password: bXlwYXNzd29yZA==  # mypassword
```

```bash
# Create secret from command line (easier)
kubectl create secret generic db-secret \
  --from-literal=username=myuser \
  --from-literal=password=mypassword

# From file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=./cert.crt \
  --from-file=tls.key=./cert.key
```

> ⚠️ **Important**: Secrets are base64 encoded, NOT encrypted by default. Use external secret managers (AWS Secrets Manager, Vault) for real security.

---

## 5. Namespaces — Virtual Clusters 🗂️

```
NAMESPACES:
──────────────────────────────────────────────────────────────

One Physical Cluster
│
├── namespace: default          ← where things go if unspecified
├── namespace: kube-system      ← K8s system components
├── namespace: kube-public      ← public cluster info
├── namespace: development      ← dev team resources
├── namespace: staging          ← staging environment
└── namespace: production       ← production resources

Benefits:
• Isolate environments (dev/staging/prod) on same cluster
• Apply resource quotas per namespace
• Control access per namespace (RBAC)
• Avoid naming conflicts
```

```bash
# Namespace commands
kubectl get namespaces
kubectl create namespace development
kubectl delete namespace development

# Run resources in namespace
kubectl apply -f deployment.yaml -n development
kubectl get pods -n development
kubectl get pods --all-namespaces   # or -A

# Set default namespace (so you don't type -n every time)
kubectl config set-context --current --namespace=development
```

---

## 6. kubectl — All Important Commands 💻

### Setup & Context

```bash
# Check version
kubectl version

# Cluster info
kubectl cluster-info

# View config (kubeconfig)
kubectl config view

# List contexts (clusters)
kubectl config get-contexts

# Switch cluster
kubectl config use-context my-cluster

# Set default namespace
kubectl config set-context --current --namespace=production
```

### Get — View Resources

```bash
# Pods
kubectl get pods                          # default namespace
kubectl get pods -n kube-system           # specific namespace
kubectl get pods -A                       # all namespaces
kubectl get pods -o wide                  # show node and IP
kubectl get pods -o yaml                  # full YAML output
kubectl get pods -w                       # watch (live updates)
kubectl get pods -l app=my-app            # filter by label

# All resource types
kubectl get all                           # pods, services, deployments
kubectl get deployments
kubectl get services
kubectl get configmaps
kubectl get secrets
kubectl get namespaces
kubectl get nodes
kubectl get events                        # cluster events
kubectl get events --sort-by='.lastTimestamp'
```

### Describe — Detailed Info

```bash
# Most useful for debugging!
kubectl describe pod my-pod
kubectl describe deployment my-app
kubectl describe service my-service
kubectl describe node worker-1

# Look for:
# • Events section at bottom ← shows what happened
# • Conditions section ← shows health
# • Volumes section ← shows mounts
```

### Apply & Delete

```bash
# Create/update from file
kubectl apply -f deployment.yaml
kubectl apply -f ./k8s/                   # entire directory
kubectl apply -f https://url/file.yaml    # from URL

# Delete
kubectl delete -f deployment.yaml
kubectl delete pod my-pod
kubectl delete deployment my-app
kubectl delete service my-service
kubectl delete all --all                  # delete everything in namespace
```

### Logs & Debugging

```bash
# View logs
kubectl logs my-pod
kubectl logs -f my-pod                    # follow (like tail -f)
kubectl logs my-pod --tail=100            # last 100 lines
kubectl logs my-pod -c container-name     # specific container
kubectl logs my-pod --previous            # previous crashed container ← very useful!

# Execute commands in pod
kubectl exec -it my-pod -- bash
kubectl exec -it my-pod -- sh            # if bash not available
kubectl exec my-pod -- ls /app
kubectl exec -it my-pod -c sidecar -- bash  # specific container

# Copy files
kubectl cp my-pod:/app/logs ./logs        # from pod to local
kubectl cp ./config.yaml my-pod:/app/     # local to pod

# Port forward (access pod locally)
kubectl port-forward my-pod 8080:3000
kubectl port-forward service/my-service 8080:80
# Now open http://localhost:8080
```

### Scaling & Updates

```bash
# Scale deployment
kubectl scale deployment my-app --replicas=5
kubectl scale deployment my-app --replicas=1

# Update image (rolling update)
kubectl set image deployment/my-app my-app=myrepo/my-app:2.0.0

# Watch rollout
kubectl rollout status deployment/my-app

# Rollout history
kubectl rollout history deployment/my-app

# Rollback to previous version
kubectl rollout undo deployment/my-app

# Rollback to specific version
kubectl rollout undo deployment/my-app --to-revision=2

# Pause/resume rollout
kubectl rollout pause deployment/my-app
kubectl rollout resume deployment/my-app
```

### Resource Management

```bash
# Edit resource live
kubectl edit deployment my-app            # opens in vim/nano

# Patch resource
kubectl patch deployment my-app \
  -p '{"spec":{"replicas":3}}'

# Label resources
kubectl label pod my-pod env=production
kubectl label pod my-pod env-            # remove label

# Annotate
kubectl annotate pod my-pod description="main api pod"

# Top (resource usage — needs metrics-server)
kubectl top nodes
kubectl top pods
kubectl top pods -n production
```

---

## 7. Your First Real Kubernetes Deployment 🚀

Let's deploy a complete Node.js app on Kubernetes step by step.

### Prerequisites

```bash
# Install minikube (local K8s for learning)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start minikube
minikube start

# Verify
kubectl get nodes
# NAME       STATUS   ROLES           AGE
# minikube   Ready    control-plane   1m
```

### Project Structure

```
k8s-demo/
├── app/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
└── k8s/
    ├── namespace.yaml
    ├── configmap.yaml
    ├── secret.yaml
    ├── deployment.yaml
    └── service.yaml
```

### Step 1: The Application

```javascript
// app/app.js
const express = require('express');
const app = express();

const config = {
  appName: process.env.APP_NAME || 'K8s Demo',
  environment: process.env.NODE_ENV || 'development',
  dbHost: process.env.DB_HOST || 'localhost'
};

app.get('/', (req, res) => {
  res.json({
    message: `Hello from ${config.appName}!`,
    environment: config.environment,
    hostname: require('os').hostname(),  // shows pod name!
    version: '1.0.0'
  });
});

app.get('/health', (req, res) => res.json({ status: 'healthy' }));
app.get('/ready', (req, res) => res.json({ status: 'ready' }));

app.listen(3000, () => console.log('Server on port 3000'));
```

```dockerfile
# app/Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY app.js .
RUN addgroup -S app && adduser -S app -G app
USER app
EXPOSE 3000
HEALTHCHECK --interval=30s CMD wget -qO- http://localhost:3000/health || exit 1
CMD ["node", "app.js"]
```

```bash
# Build and push image
docker build -t your-dockerhub-username/k8s-demo:v1 ./app
docker push your-dockerhub-username/k8s-demo:v1
```

### Step 2: Namespace

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
  labels:
    env: demo
```

### Step 3: ConfigMap

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: demo
data:
  APP_NAME: "K8s Demo App"
  NODE_ENV: "production"
  LOG_LEVEL: "info"
```

### Step 4: Secret

```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
  namespace: demo
type: Opaque
stringData:              # stringData auto-encodes to base64
  DB_PASSWORD: "supersecretpassword"
  API_KEY: "my-api-key-12345"
```

### Step 5: Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-demo
  namespace: demo
  labels:
    app: k8s-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: k8s-demo

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  template:
    metadata:
      labels:
        app: k8s-demo
    spec:
      containers:
        - name: k8s-demo
          image: your-dockerhub-username/k8s-demo:v1
          ports:
            - containerPort: 3000

          resources:
            requests:
              memory: "64Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"

          envFrom:
            - configMapRef:
                name: app-config

          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secret
                  key: DB_PASSWORD

          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10

          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
```

### Step 6: Service

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: k8s-demo-service
  namespace: demo
spec:
  selector:
    app: k8s-demo
  type: NodePort           # accessible from outside minikube
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30080      # access at minikube-ip:30080
```

### Step 7: Deploy Everything!

```bash
# Apply all files in order
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Or apply entire directory at once
kubectl apply -f k8s/

# Watch pods come up
kubectl get pods -n demo -w

# Check everything
kubectl get all -n demo

# Get minikube IP
minikube ip

# Test the app!
curl http://$(minikube ip):30080/
# or
minikube service k8s-demo-service -n demo --url
```

### Step 8: Test Self-Healing!

```bash
# Delete a pod — watch K8s recreate it!
kubectl delete pod <pod-name> -n demo

# Watch pods
kubectl get pods -n demo -w
# Pod disappears → new one appears in seconds ✅

# Scale up
kubectl scale deployment k8s-demo --replicas=5 -n demo
kubectl get pods -n demo

# Rolling update
kubectl set image deployment/k8s-demo k8s-demo=your-username/k8s-demo:v2 -n demo
kubectl rollout status deployment/k8s-demo -n demo

# Rollback if needed
kubectl rollout undo deployment/k8s-demo -n demo
```

---

## 8. Kubernetes YAML Writing Guide 📝

```yaml
# Every K8s resource has these 4 top-level fields:

apiVersion: apps/v1        # API version (check docs for correct version)
kind: Deployment           # resource type
metadata:                  # info about the resource
  name: my-app             # required
  namespace: default       # optional (default if not set)
  labels:                  # key-value pairs for selecting
    app: my-app
    env: production
  annotations:             # non-identifying metadata
    description: "Main API"
spec:                      # desired state (varies by resource type)
  ...

# API versions cheatsheet:
# Pod, Service, ConfigMap, Secret, Namespace → v1
# Deployment, ReplicaSet, DaemonSet          → apps/v1
# Ingress                                    → networking.k8s.io/v1
# HorizontalPodAutoscaler                    → autoscaling/v2
# ClusterRole, RoleBinding                   → rbac.authorization.k8s.io/v1
```

---

## 9. Common Errors & Fixes 🔧

```bash
# CrashLoopBackOff — container keeps crashing
kubectl logs pod-name --previous    # see crash logs
kubectl describe pod pod-name       # check events
# Fix: check app logs for startup errors

# ImagePullBackOff — can't pull image
kubectl describe pod pod-name       # see error message
# Fix: check image name/tag, registry credentials

# Pending — pod can't be scheduled
kubectl describe pod pod-name       # see events
# Fix: check resource requests vs node capacity

# OOMKilled — out of memory
kubectl describe pod pod-name       # shows OOMKilled
# Fix: increase memory limits

# Error: connection refused
kubectl get endpoints service-name  # check if pods are targeted
kubectl get pods -l app=my-app      # check pod labels match selector

# Check node resources
kubectl describe node node-name     # see allocated vs available
kubectl top nodes                   # live usage
```

---

## Quick Reference Cheatsheet 📋

```bash
# GET
kubectl get pods/deployments/services/nodes -n namespace
kubectl get all -A

# DESCRIBE (debugging)
kubectl describe pod/deployment/service name

# LOGS
kubectl logs -f pod-name --previous

# EXEC
kubectl exec -it pod-name -- bash

# APPLY / DELETE
kubectl apply -f file.yaml
kubectl delete -f file.yaml

# SCALE
kubectl scale deployment name --replicas=5

# UPDATE IMAGE
kubectl set image deployment/name container=image:tag

# ROLLOUT
kubectl rollout status/history/undo deployment/name

# PORT FORWARD
kubectl port-forward pod/service 8080:3000

# NAMESPACES
kubectl get pods -n namespace
kubectl config set-context --current --namespace=namespace
```

---

## What's in Part 2? 🚀

Part 1 covered the fundamentals. **Part 2** goes deeper:

- 🌐 **Ingress** — route external traffic to services
- 💾 **Persistent Volumes** — storage for stateful apps
- 📈 **HPA** — auto-scale based on CPU/memory
- 🔒 **RBAC** — control who can do what
- ⛵ **Helm** — package manager for K8s
- 🏭 **Real production project** — full app on K8s
- 🔄 **K8s + CI/CD** — auto-deploy to K8s

> 💪 **Practice**: Install minikube, deploy the demo project above. Break things on purpose — delete pods, scale up and down, watch K8s fix everything. That's how you really learn K8s!