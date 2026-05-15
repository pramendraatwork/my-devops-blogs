---
title: "Kubernetes — From Zero to First Deployment"
date: 2024-01-25
draft: false
description: "K8s core objects with real YAML. Pods, Deployments, Services and kubectl cheatsheet."
categories: ["kubernetes"]
tags: ["kubernetes", "k8s", "kubectl", "pods"]
showToc: true
---
You declare **what you want**. K8s makes it happen and keeps it that way.

## kubectl cheatsheet

```bash
kubectl get pods
kubectl get pods -A
kubectl describe pod my-pod
kubectl logs -f my-pod
kubectl exec -it my-pod -- bash
kubectl apply -f deployment.yaml
kubectl delete -f deployment.yaml
kubectl scale deployment my-app --replicas=5
kubectl rollout undo deployment/my-app
```

## Deployment manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: myrepo/my-app:1.0.0
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /healthz
              port: 3000
            initialDelaySeconds: 15
```

## Service manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 3000
```

## Things that got me

- Pods are ephemeral — always use a Deployment
- Selector MUST match template labels — number 1 YAML mistake
- CrashLoopBackOff → check kubectl logs pod-name --previous
- ImagePullBackOff → wrong image name/tag or missing registry creds
