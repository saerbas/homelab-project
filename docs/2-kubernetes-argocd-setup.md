# Phase 2: Kubernetes & ArgoCD Setup

## 1. Overview
**Goal:** Deploy a containerized nginx application on K3s using GitOps via ArgoCD

**Components:**
* **ArgoCD:** Continuous Deployment controller – watches Git repo for changes
* **Deployment:** 5 nginx replicas for load distribution
* **Service:** ClusterIP to expose nginx internally
* **Ingress:** Route external traffic via nginx.local domain

---

## 2. Architecture

### ArgoCD Application (argocd-app.yml)
The entry point for GitOps automation.

```yaml
source:
  repoURL: https://github.com/saerbas/homelab-project.git
  targetRevision: main
  path: k8s/nginx-demo
```
- **Watches:** GitHub repository for changes in k8s/nginx-demo
- **Auto-syncs:** Applies changes automatically when detected
- **Self-healing:** Reverts manual cluster changes back to Git state
- **Pruning:** Deletes resources removed from Git

---

## 3. Kubernetes Resources

### Deployment (deployment.yml)
Manages 5 nginx replicas with resource limits.

| Setting | Value |
|---------|-------|
| Replicas | 5 |
| Image | nginx:latest |
| Memory Request | 64Mi |
| Memory Limit | 128Mi |
| CPU Request | 250m |
| CPU Limit | 500m |

**Why 5 replicas?**
- High availability: If one pod fails, 4 continue running
- Load distribution: Requests spread across pods

### Service (service.yml)
Exposes nginx internally via ClusterIP.

```
Client → Ingress (nginx.local) → Service (port 80) → Deployment Pods
```

- **Type:** ClusterIP (internal only, not exposed to outside)
- **Selector:** Routes to pods with label `app: mein-nginx`
- **Port Mapping:** Service port 80 → Container port 80

### Ingress (ingress.yml)
Routes external requests to the service.

```
External Request (nginx.local) → Ingress Rule → Service → Pods
```

- **Domain:** nginx.local
- **Path:** / (root path)
- **Backend:** nginx-service on port 80

---

## 4. GitOps Workflow

### How it works:
1. You commit changes to `k8s/nginx-demo/` in Git
2. ArgoCD detects changes (every 3 minutes by default)
3. ArgoCD applies the new configuration to K3s
4. Kubernetes schedules the pods
5. Ingress routes traffic to the service

### Benefits:
- **Single Source of Truth:** Git repo defines desired state
- **Audit Trail:** Every change tracked in Git history
- **Rollback:** Revert to any previous Git commit
- **No Manual Kubectl:** Infrastructure changes via Git only

---

## 5. Testing & Verification

```bash
# Check ArgoCD application status
argocd app get mein-nginx-projekt

# View deployed pods
kubectl get pods -n default

# Test nginx access
curl http://nginx.local/

# View resource usage
kubectl top nodes
kubectl top pods
```

---

## 6. Key Concepts

**Idempotency:** Running the same deployment multiple times yields the same result.

**Declarative:** You declare desired state (YAML files), K8s makes it happen.

**Self-Healing:** If a pod crashes, K8s automatically restarts it.

**Scalability:** Change `replicas: 5` to `replicas: 10` → ArgoCD applies it automatically.

**GitOps:** Infrastructure = Code. Version control = Infrastructure history.
