# Kubernetes Deployments - Complete Guide

## 📌 What is a Deployment?

A **Deployment** is a Kubernetes resource that enables **declarative updates for Pods and ReplicaSets**. It's one of the most commonly used workload resources in Kubernetes.

**In simple terms:**
- A Deployment tells Kubernetes: *"I want X copies of my application running, and here's how to create them"*
- Kubernetes ensures the desired state is always maintained
- If a Pod crashes, the Deployment automatically replaces it

---

## 🔶 Why Use Deployments?

| Feature | Description |
|---------|-------------|
| **Declarative Updates** | Define the desired state, Kubernetes handles the rest |
| **Rolling Updates** | Update applications with zero downtime |
| **Rollback** | Easily revert to previous versions |
| **Scaling** | Scale up/down with a simple command |
| **Self-Healing** | Automatically replaces failed Pods |
| **Version History** | Maintains history for rollbacks |

---

## 🔶 Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          DEPLOYMENT                                  │
│                    (nginx-deployment)                                │
│                                                                      │
│   ┌───────────────────────────────────────────────────────────────┐ │
│   │                      ReplicaSet                                │ │
│   │              (nginx-deployment-5d4f6bc9f)                      │ │
│   │                                                                │ │
│   │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐           │ │
│   │   │    Pod 1    │  │    Pod 2    │  │    Pod 3    │           │ │
│   │   │   (nginx)   │  │   (nginx)   │  │   (nginx)   │           │ │
│   │   └─────────────┘  └─────────────┘  └─────────────┘           │ │
│   │                                                                │ │
│   └───────────────────────────────────────────────────────────────┘ │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Hierarchy:**
1. **Deployment** → Manages ReplicaSets
2. **ReplicaSet** → Ensures a specific number of Pod replicas
3. **Pods** → Run your actual containers

---

## 🔶 Deployment YAML Structure

A Deployment YAML file has **four main sections**:

```yaml
apiVersion: apps/v1      # API version
kind: Deployment         # Resource type
metadata:                # Deployment metadata
  name: my-app
spec:                    # Deployment specification
  replicas: 3
  selector: ...
  template: ...          # Pod template
```

---

## 📄 Complete Deployment YAML Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    environment: production
spec:
  # Number of Pod replicas
  replicas: 3
  
  # How to find Pods that belong to this Deployment
  selector:
    matchLabels:
      app: nginx
  
  # Deployment strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  
  # Pod readiness configuration
  minReadySeconds: 5
  
  # Rollback history
  revisionHistoryLimit: 10
  
  # Progress deadline
  progressDeadlineSeconds: 600
  
  # Pod template (what Pods look like)
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## 📖 Field-by-Field Explanation

### 1. apiVersion & kind

```yaml
apiVersion: apps/v1
kind: Deployment
```

| Field | Value | Description |
|-------|-------|-------------|
| `apiVersion` | `apps/v1` | The API group and version for Deployments |
| `kind` | `Deployment` | The type of Kubernetes resource |

---

### 2. metadata

```yaml
metadata:
  name: nginx-deployment
  labels:
    app: nginx
    environment: production
```

| Field | Description |
|-------|-------------|
| `name` | **Required.** Unique name for the Deployment |
| `labels` | Key-value pairs to organize and select resources |
| `annotations` | Non-identifying metadata (optional) |
| `namespace` | Namespace where Deployment lives (defaults to "default") |

---

### 3. spec.replicas

```yaml
spec:
  replicas: 3
```

| Field | Default | Description |
|-------|---------|-------------|
| `replicas` | 1 | Number of Pod copies to maintain |

---

### 4. spec.selector (Required)

```yaml
selector:
  matchLabels:
    app: nginx
```

> **Critical:** The selector **MUST match** the labels in `template.metadata.labels`

| Field | Description |
|-------|-------------|
| `matchLabels` | Simple key-value label matching |
| `matchExpressions` | Advanced expressions (In, NotIn, Exists, DoesNotExist) |

---

### 5. spec.strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 25%
    maxUnavailable: 25%
```

**Two Strategy Types:**

| Type | Description |
|------|-------------|
| `RollingUpdate` | **(Default)** Gradually replaces old Pods with new ones |
| `Recreate` | Kills all existing Pods before creating new ones |

**RollingUpdate Parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `maxSurge` | 25% | Max Pods that can exist above desired count during update |
| `maxUnavailable` | 25% | Max Pods that can be unavailable during update |

**Visual Example (10 replicas, maxSurge=25%, maxUnavailable=25%):**

```
During update:
- Maximum Pods running: 10 + 25% = 13 Pods
- Minimum Pods available: 10 - 25% = 8 Pods

Step 1: Start 3 new Pods    → 13 Pods (10 old + 3 new)
Step 2: Terminate 3 old      → 10 Pods (7 old + 3 new)
Step 3: Start 3 new         → 13 Pods (7 old + 6 new)
Step 4: Terminate 3 old     → 10 Pods (4 old + 6 new)
Step 5: Continue until done...
```

---

### 6. spec.minReadySeconds

```yaml
minReadySeconds: 5
```

| Field | Default | Description |
|-------|---------|-------------|
| `minReadySeconds` | 0 | Seconds a Pod must be ready without crashing to be considered available |

---

### 7. spec.revisionHistoryLimit

```yaml
revisionHistoryLimit: 10
```

| Field | Default | Description |
|-------|---------|-------------|
| `revisionHistoryLimit` | 10 | Number of old ReplicaSets to retain for rollback |

---

### 8. spec.progressDeadlineSeconds

```yaml
progressDeadlineSeconds: 600
```

| Field | Default | Description |
|-------|---------|-------------|
| `progressDeadlineSeconds` | 600 | Max seconds for deployment to progress before considered failed |

---

### 9. spec.template (Required)

This is the **Pod template** - defines how Pods should be created:

```yaml
template:
  metadata:
    labels:
      app: nginx           # MUST match selector.matchLabels
  spec:
    containers:
    - name: nginx
      image: nginx:1.21
      ports:
      - containerPort: 80
```

---

## 🔄 Deployment Status Fields

When you run `kubectl get deployment -o yaml`, you'll see status information:

| Field | Description |
|-------|-------------|
| `replicas` | Total Pods targeted by this deployment |
| `availableReplicas` | Pods available (ready for minReadySeconds) |
| `readyReplicas` | Pods with Ready condition |
| `unavailableReplicas` | Pods still required for 100% capacity |
| `updatedReplicas` | Pods with the desired template spec |
| `conditions` | Current state observations |

---

## 🛠️ Common kubectl Commands for Deployments

### Create & Apply

```bash
# Create from YAML
kubectl apply -f deployment.yaml

# Create imperatively (not recommended for production)
kubectl create deployment nginx --image=nginx:1.21 --replicas=3
```

### View & Inspect

```bash
# List all deployments
kubectl get deployments

# Detailed view
kubectl get deployment nginx-deployment -o wide

# Full YAML output
kubectl get deployment nginx-deployment -o yaml

# Describe (detailed info + events)
kubectl describe deployment nginx-deployment
```

### Scaling

```bash
# Scale to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# Autoscale (requires metrics-server)
kubectl autoscale deployment nginx-deployment --min=3 --max=10 --cpu-percent=80
```

### Update Image

```bash
# Update container image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Record the change for rollback
kubectl set image deployment/nginx-deployment nginx=nginx:1.22 --record
```

### Rollout Management

```bash
# Check rollout status
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment

# View specific revision
kubectl rollout history deployment/nginx-deployment --revision=2

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Pause rollout
kubectl rollout pause deployment/nginx-deployment

# Resume rollout
kubectl rollout resume deployment/nginx-deployment

# Restart deployment (rolling restart)
kubectl rollout restart deployment/nginx-deployment
```

### Delete

```bash
# Delete deployment
kubectl delete deployment nginx-deployment

# Delete from YAML
kubectl delete -f deployment.yaml
```

---

## 📝 Step-by-Step: Creating a Deployment YAML File

### Step 1: Start with the Basic Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app:v1
```

### Step 2: Add Replicas

```yaml
spec:
  replicas: 3
```

### Step 3: Add Resource Limits

```yaml
containers:
- name: my-app
  image: my-app:v1
  resources:
    requests:
      memory: "64Mi"
      cpu: "250m"
    limits:
      memory: "128Mi"
      cpu: "500m"
```

### Step 4: Add Health Checks

```yaml
containers:
- name: my-app
  image: my-app:v1
  livenessProbe:
    httpGet:
      path: /healthz
      port: 8080
    initialDelaySeconds: 15
    periodSeconds: 10
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
```

### Step 5: Add Update Strategy

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### Step 6: Apply the Deployment

```bash
kubectl apply -f deployment.yaml
```

---

## 📋 Minimal Deployment Template

Use this as a quick starting point:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
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
      - name: my-app
        image: my-app:1.0
        ports:
        - containerPort: 80
```

---

## ✅ Best Practices

1. **Always set resource requests and limits** for containers
2. **Use health checks** (liveness & readiness probes)
3. **Use labels consistently** across all resources
4. **Set `revisionHistoryLimit`** appropriately (don't keep too many)
5. **Use Rolling Update strategy** for zero-downtime deployments
6. **Never use `latest` tag** for production images
7. **Store YAML files in version control**
8. **Use namespaces** to organize deployments

---

## 🔗 References

- [Official Kubernetes Documentation - Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Deployment API Reference](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
