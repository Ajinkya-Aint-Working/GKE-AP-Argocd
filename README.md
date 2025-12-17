# Argo CD on GKE Autopilot (Cost-Optimized Setup)

This repository documents how to deploy **Argo CD** on **Google Kubernetes Engine (GKE) Autopilot** with **custom CPU and memory requests/limits** to reduce costs.

By default, GKE Autopilot assigns **0.5 vCPU and 2GB RAM per container**, which can significantly increase cost.  
This setup overrides those defaults with minimal, production-safe resource values for each Argo CD component.

---

## Why Custom Resources Are Required in Autopilot

GKE Autopilot:
- Automatically enforces **minimum resource defaults**
- Charges **per requested resources**, not actual usage
- Defaults to **500m CPU / 2Gi memory** if not specified

Argo CD components are **lightweight** and do not need those defaults.

This configuration:
- Explicitly defines CPU & memory for every component
- Prevents Autopilot from inflating resource usage
- Reduces overall cluster cost

---

## Resource Strategy Overview

| Component | CPU | Memory |
|---------|-----|--------|
| Init Containers (global) | 150m | 256Mi |
| argocd-server | 150m | 256Mi |
| repo-server | 250m | 512Mi |
| application-controller | 200m | 512Mi |
| redis | 100m | 256Mi |
| application-set-controller | 100m | 128Mi |
| notifications-controller | 50m | 128Mi |
| dex | 100m | 128Mi |

---

## values.yaml (Cost-Optimized)

```yaml
global:
  resources:
    requests:
      cpu: 150m
      memory: 256Mi
    limits:
      cpu: 150m
      memory: 256Mi

configs:
  params:
    server.insecure: true

server:
  resources:
    requests:
      cpu: 150m
      memory: 256Mi
    limits:
      cpu: 150m
      memory: 256Mi

repoServer:
  resources:
    requests:
      cpu: 250m
      memory: 512Mi
    limits:
      cpu: 250m
      memory: 512Mi

controller:
  resources:
    requests:
      cpu: 200m
      memory: 512Mi
    limits:
      cpu: 200m
      memory: 512Mi

redis:
  enabled: true
  resources:
    requests:
      cpu: 100m
      memory: 256Mi
    limits:
      cpu: 100m
      memory: 256Mi

applicationSet:
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 100m
      memory: 128Mi

notifications:
  resources:
    requests:
      cpu: 50m
      memory: 128Mi
    limits:
      cpu: 50m
      memory: 128Mi

dex:
  enabled: true
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 100m
      memory: 128Mi
```

---

## Installation Steps

```bash
kubectl create namespace argocd
```

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

```bash
helm install argocd argo/argo-cd \
  --version 9.1.8 \
  -n argocd \
  -f values.yaml
```

---

## Expose Argo CD UI

```bash
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

---

## Get Admin Password

```bash
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 --decode && echo
```

---

## Access UI

```
https://<EXTERNAL-IP>
```

---

##  Check the Resources and limits


```
ubectl get pods -n argocd \
  -o=custom-columns=PO:.metadata.name,CONTAINER:.spec.containers[*].name,\
REQ_CPU:.spec.containers[*].resources.requests.cpu,REQ_MEM:.spec.containers[*].resources.requests.memory,\
LIM_CPU:.spec.containers[*].resources.limits.cpu,LIM_MEM:.spec.containers[*].resources.limits.memory

```

