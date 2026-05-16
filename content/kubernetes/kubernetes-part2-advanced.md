---
title: "Kubernetes Part 2 — Intermediate to Advanced Guide"
date: 2024-03-15
draft: false
description: "Advanced Kubernetes: Ingress, Persistent Volumes, HPA auto-scaling, RBAC, Helm, StatefulSets, and a complete production project with CI/CD."
categories: ["kubernetes"]
tags: ["kubernetes", "k8s", "helm", "ingress", "hpa", "rbac", "advanced", "devops"]
showToc: true
TocOpen: true
---

## 1. Ingress — Route External Traffic 🌐

A Service of type `LoadBalancer` gives you one IP per service — expensive and messy. **Ingress** is a single entry point that routes traffic to multiple services based on URL path or hostname.

```
WITHOUT INGRESS:
──────────────────────────────────────────────────────────────
api.example.com    → LoadBalancer ($$) → API Service
app.example.com    → LoadBalancer ($$) → Frontend Service
admin.example.com  → LoadBalancer ($$) → Admin Service
3 load balancers = 3x the cost! 💸

WITH INGRESS:
──────────────────────────────────────────────────────────────
                    ┌─────────────────────────┐
Internet ──────────▶│    Ingress Controller   │
                    │    (ONE Load Balancer)  │
                    └────────────┬────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │       Ingress Rules      │
                    │                         │
                    │ /api/*   → API Service  │
                    │ /app/*   → Frontend     │
                    │ /admin/* → Admin        │
                    └─────────────────────────┘

One load balancer, multiple services! ✅
```

### Setting Up Ingress

```bash
# Install NGINX Ingress Controller (minikube)
minikube addons enable ingress

# Or with helm
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

### Path-Based Routing

```yaml
# ingress-path.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: production
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80

          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### Host-Based Routing

```yaml
# ingress-host.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - api.example.com
        - app.example.com
      secretName: tls-secret      # TLS certificate

  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80

    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

### TLS/HTTPS with cert-manager

```bash
# Install cert-manager (auto SSL from Let's Encrypt)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.0/cert-manager.yaml

# Create ClusterIssuer
cat << EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: pramendraatwork@gmail.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
EOF
```

```yaml
# Ingress with auto TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls      # cert-manager creates this!
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

---

## 2. Persistent Volumes — Storage for Stateful Apps 💾

Pods are ephemeral — when they die, their data dies. For databases and stateful apps, you need storage that survives pod restarts.

```
KUBERNETES STORAGE HIERARCHY:
──────────────────────────────────────────────────────────────

PersistentVolume (PV)
  • Actual storage resource (disk, NFS, cloud storage)
  • Created by cluster admin
  • Like a physical hard drive

PersistentVolumeClaim (PVC)
  • Request for storage by a pod
  • Like requesting a hard drive of certain size
  • Binds to a matching PV

StorageClass
  • Defines how to dynamically provision storage
  • "Slow" (HDD), "Fast" (SSD), "Premium" (NVMe)
  • Cloud providers have built-in storage classes

FLOW:
Developer creates PVC → K8s finds matching PV → Pod mounts PVC
```

### StorageClass (Dynamic Provisioning)

```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs    # AWS EBS
parameters:
  type: gp3
  fsType: ext4
  encrypted: "true"
reclaimPolicy: Retain                 # keep disk when PVC deleted
allowVolumeExpansion: true            # allow resizing
volumeBindingMode: WaitForFirstConsumer
```

### PersistentVolumeClaim

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce          # one node at a time (RWO)
    # ReadOnlyMany  (ROX)   → many nodes, read-only
    # ReadWriteMany (RWX)   → many nodes, read-write (needs NFS/EFS)
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 20Gi
```

### Using PVC in a Pod

```yaml
# postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: production
spec:
  replicas: 1              # databases usually single replica!
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
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata

          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data

          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"

      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc    # use our PVC!
```

---

## 3. StatefulSets — For Stateful Applications ⚡

Deployments are great for stateless apps. For databases and stateful apps, use **StatefulSets**.

```
DEPLOYMENT vs STATEFULSET:
──────────────────────────────────────────────────────────────

DEPLOYMENT:
• Pods: my-app-7d9f8b-xkq2p (random names)
• Pods are interchangeable
• No guaranteed ordering
• Shared or no storage
• Good for: APIs, web servers

STATEFULSET:
• Pods: postgres-0, postgres-1, postgres-2 (stable names!)
• Each pod has unique identity
• Ordered creation/deletion (0 → 1 → 2)
• Each pod gets its own PVC
• Good for: databases, Kafka, Zookeeper, Redis cluster
```

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: production
spec:
  serviceName: postgres          # headless service name
  replicas: 3
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
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data

  # Each pod gets its OWN PVC automatically!
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-ssd
        resources:
          requests:
            storage: 10Gi

---
# Headless service for StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: production
spec:
  clusterIP: None              # headless!
  selector:
    app: postgres
  ports:
    - port: 5432
# Pods accessible as:
# postgres-0.postgres.production.svc.cluster.local
# postgres-1.postgres.production.svc.cluster.local
```

---

## 4. HPA — Horizontal Pod Autoscaler 📈

HPA automatically scales your deployment based on CPU/memory usage or custom metrics.

```
HPA IN ACTION:
──────────────────────────────────────────────────────────────

Normal traffic (CPU 20%):
Deployment: [Pod 1] [Pod 2]    ← 2 replicas

Traffic spike (CPU 80%):
HPA detects: CPU > 70% threshold!
Deployment: [Pod 1] [Pod 2] [Pod 3] [Pod 4] [Pod 5]  ← scales up!

Traffic drops (CPU 15%):
HPA detects: CPU < 30% for 5 minutes
Deployment: [Pod 1] [Pod 2]    ← scales back down!
```

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  minReplicas: 2              # always at least 2
  maxReplicas: 20             # never more than 20

  metrics:
    # Scale on CPU
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70    # scale when avg CPU > 70%

    # Scale on Memory
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80    # scale when avg Memory > 80%

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60      # wait 60s before scaling up again
      policies:
        - type: Pods
          value: 4                        # add max 4 pods at a time
          periodSeconds: 60

    scaleDown:
      stabilizationWindowSeconds: 300     # wait 5min before scaling down
      policies:
        - type: Pods
          value: 2                        # remove max 2 pods at a time
          periodSeconds: 60
```

```bash
# Check HPA status
kubectl get hpa -n production
kubectl describe hpa my-app-hpa -n production

# Simulate load to trigger scaling
kubectl run load-test --image=busybox --rm -it -- \
  sh -c "while true; do wget -q -O- http://my-app-service/; done"

# Watch pods scale
kubectl get pods -n production -w
```

---

## 5. RBAC — Role Based Access Control 🔒

RBAC controls who can do what in your cluster.

```
RBAC CONCEPTS:
──────────────────────────────────────────────────────────────

ServiceAccount  → Identity (who am I?)
Role            → Permissions (what can I do?) in a namespace
ClusterRole     → Permissions across all namespaces
RoleBinding     → Connect ServiceAccount to Role
ClusterRoleBinding → Connect ServiceAccount to ClusterRole

EXAMPLE:
Developer John can only:
  • get/list/watch pods in "development" namespace
  • NOT delete pods
  • NOT access "production" namespace
```

### Creating RBAC

```yaml
# 1. ServiceAccount — identity for pod/user
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer-sa
  namespace: development

---
# 2. Role — what permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: development
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps"]
    verbs: ["get", "list", "watch"]      # read only

  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch", "update"]  # can update deployments

  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list"]              # can view logs

  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create"]                   # can exec into pods

---
# 3. RoleBinding — connect them
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: development
subjects:
  - kind: ServiceAccount
    name: developer-sa
    namespace: development
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

### Common RBAC Patterns

```yaml
# Read-only access to everything in a namespace
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["get", "list", "watch"]

# Admin access to a namespace
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]

# CI/CD pipeline service account
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "update", "patch"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["get", "list"]
```

```bash
# Check if a user can do something
kubectl auth can-i get pods --namespace=production
kubectl auth can-i delete deployments --as=developer-sa

# List roles
kubectl get roles -n development
kubectl get clusterroles

# List bindings
kubectl get rolebindings -n development
kubectl describe rolebinding developer-binding -n development
```

---

## 6. Helm — Package Manager for Kubernetes ⛵

Helm is like `apt` or `npm` but for Kubernetes. Instead of managing 10 YAML files separately, Helm bundles them into a **chart**.

```
WITHOUT HELM:
──────────────────────────────────────────────────────────────
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f hpa.yaml
→ 7 separate commands, 7 files to manage, no versioning!

WITH HELM:
──────────────────────────────────────────────────────────────
helm install my-app ./my-chart
→ One command deploys everything!
→ Versioned, rollback-able, configurable!
```

### Installing Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### Using Existing Charts

```bash
# Add chart repositories
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Search for charts
helm search repo postgres
helm search repo nginx

# Install a chart
helm install my-postgres bitnami/postgresql \
  --namespace production \
  --create-namespace \
  --set auth.postgresPassword=secretpassword \
  --set primary.persistence.size=20Gi

# Install with values file
helm install my-postgres bitnami/postgresql \
  --namespace production \
  -f values.yaml

# List installed releases
helm list -A

# Upgrade
helm upgrade my-postgres bitnami/postgresql \
  --namespace production \
  --set auth.postgresPassword=newpassword

# Rollback
helm rollback my-postgres 1    # rollback to revision 1

# Uninstall
helm uninstall my-postgres -n production
```

### Creating Your Own Chart

```bash
# Create chart structure
helm create my-app

# Structure:
my-app/
├── Chart.yaml          # chart metadata
├── values.yaml         # default values
├── templates/          # K8s manifests with templating
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl    # reusable template functions
└── charts/             # sub-charts (dependencies)
```

```yaml
# values.yaml — configurable defaults
replicaCount: 3

image:
  repository: myrepo/my-app
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  host: myapp.example.com

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
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

```bash
# Install your chart
helm install my-app ./my-app -n production

# Override values at install
helm install my-app ./my-app \
  --set replicaCount=5 \
  --set image.tag=2.0.0

# Validate before installing
helm lint ./my-app
helm template my-app ./my-app    # render templates locally
helm install --dry-run my-app ./my-app  # dry run
```

---

## 7. Resource Quotas & Limits 📊

Prevent one team from consuming all cluster resources.

```yaml
# namespace-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    # Pod limits
    pods: "50"
    # Compute limits
    requests.cpu: "10"
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    # Storage limits
    requests.storage: "100Gi"
    # Object limits
    count/deployments.apps: "20"
    count/services: "20"
    count/secrets: "50"

---
# LimitRange — default limits for pods that don't specify
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
    - type: Container
      default:
        cpu: "200m"
        memory: "256Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "2Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
```

---

## 8. Production Project — Full Stack on Kubernetes 🏭

Complete production-ready deployment: React + Node.js API + PostgreSQL + Redis + Ingress.

### Project Structure

```
production-k8s/
├── namespace.yaml
├── secrets.yaml
├── configmap.yaml
├── database/
│   ├── postgres-pvc.yaml
│   ├── postgres-statefulset.yaml
│   └── postgres-service.yaml
├── cache/
│   ├── redis-deployment.yaml
│   └── redis-service.yaml
├── backend/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   └── backend-hpa.yaml
├── frontend/
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   └── frontend-hpa.yaml
└── ingress.yaml
```

### Deploy Script

```bash
#!/bin/bash
# deploy.sh
set -e

NAMESPACE="production"
IMAGE_TAG=${1:-latest}

echo "🚀 Deploying to Kubernetes..."

# Apply in order
kubectl apply -f namespace.yaml
kubectl apply -f secrets.yaml
kubectl apply -f configmap.yaml

# Database
kubectl apply -f database/
kubectl rollout status statefulset/postgres -n $NAMESPACE

# Cache
kubectl apply -f cache/
kubectl rollout status deployment/redis -n $NAMESPACE

# Backend
kubectl set image deployment/backend \
  backend=myrepo/backend:$IMAGE_TAG \
  -n $NAMESPACE
kubectl rollout status deployment/backend -n $NAMESPACE

# Frontend
kubectl set image deployment/frontend \
  frontend=myrepo/frontend:$IMAGE_TAG \
  -n $NAMESPACE
kubectl rollout status deployment/frontend -n $NAMESPACE

# Ingress
kubectl apply -f ingress.yaml

echo "✅ Deployment complete!"
kubectl get all -n $NAMESPACE
```

---

## 9. Kubernetes + CI/CD Pipeline 🔄

```yaml
# .github/workflows/deploy-k8s.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Login to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${{ secrets.KUBECONFIG }}

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/backend \
            backend=ghcr.io/${{ github.repository }}:${{ github.sha }} \
            -n production

          kubectl rollout status deployment/backend \
            -n production \
            --timeout=5m

      - name: Verify deployment
        run: |
          kubectl get pods -n production
          kubectl get ingress -n production

      - name: Notify on failure
        if: failure()
        run: |
          kubectl rollout undo deployment/backend -n production
          echo "❌ Deployment failed! Rolling back..."
```

---

## 10. Kubernetes Troubleshooting 🔧

```bash
# Pod issues
kubectl describe pod <name> -n <ns>    # check Events section!
kubectl logs <pod> --previous          # crashed container logs
kubectl logs <pod> -c <container>      # specific container

# Node issues
kubectl describe node <node-name>      # check conditions
kubectl top nodes                      # resource usage
kubectl get events --sort-by='.lastTimestamp' -A

# Network issues
kubectl exec -it pod -- curl http://service-name   # test service
kubectl exec -it pod -- nslookup service-name      # DNS check
kubectl get endpoints service-name                 # check pod targeting

# Storage issues
kubectl get pvc -n namespace           # check PVC status
kubectl describe pvc pvc-name          # see binding info
kubectl get pv                         # check PV availability

# Common error codes:
# CrashLoopBackOff  → app crashing, check logs --previous
# ImagePullBackOff  → image not found or no registry creds
# Pending           → not enough resources or no matching node
# OOMKilled         → out of memory, increase limits
# Evicted           → node ran out of resources
# Error             → check describe and logs

# Useful debugging pod
kubectl run debug --rm -it \
  --image=busybox \
  --restart=Never \
  -- sh
# Now test network, DNS, connectivity from inside cluster
```

---

## 11. Advanced Cheatsheet 📋

```bash
# INGRESS
kubectl get ingress -A
kubectl describe ingress name -n ns

# VOLUMES
kubectl get pv
kubectl get pvc -n ns
kubectl describe pvc name -n ns

# HPA
kubectl get hpa -A
kubectl describe hpa name -n ns

# RBAC
kubectl get roles,rolebindings -n ns
kubectl auth can-i get pods -n ns --as=serviceaccount:ns:sa-name

# HELM
helm list -A
helm install name ./chart -n ns
helm upgrade name ./chart -n ns
helm rollback name 1 -n ns
helm uninstall name -n ns

# RESOURCE USAGE
kubectl top nodes
kubectl top pods -A --sort-by=memory
kubectl top pods -A --sort-by=cpu

# EVENTS (great for debugging)
kubectl get events -n ns --sort-by='.lastTimestamp'
kubectl get events -n ns --field-selector reason=Failed

# LABELS
kubectl get pods -l app=my-app,env=prod
kubectl label pod my-pod env=prod
kubectl get pods --show-labels

# FORCE DELETE (stuck pods)
kubectl delete pod my-pod --force --grace-period=0

# DRAIN NODE (maintenance)
kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data
kubectl uncordon node-1    # bring back

# CORDON (stop scheduling on node)
kubectl cordon node-1
kubectl uncordon node-1
```

---

## What's Next? 🚀

You've mastered Kubernetes! The DevOps journey continues with:

- **AWS EKS** — managed Kubernetes on AWS
- **Monitoring** — Prometheus + Grafana for K8s
- **Service Mesh** — Istio for advanced traffic management
- **GitOps** — ArgoCD for declarative K8s deployments
- **Security** — Falco, OPA Gatekeeper, Pod Security

> 💪 **Real challenge**: Take the production project from Part 1, add Ingress, HPA, and deploy it with a CI/CD pipeline. Then write about it on your blog — that becomes a portfolio project that impresses interviewers!