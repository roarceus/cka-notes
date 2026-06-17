# Deployments

## What They Do

A Deployment is a **higher-level abstraction** that manages ReplicaSets and pods. It's what you use for real workloads in production.

Deployment → creates and manages → ReplicaSet → creates and manages → Pods

Extra capabilities over ReplicaSet:
- **Rolling updates** — update pods gradually, zero downtime
- **Rollbacks** — revert to a previous version instantly
- **Pause & resume** — batch multiple changes before applying

---

## Deployment YAML

Almost identical to a ReplicaSet — only `kind` changes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  template:
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

---

## Essential Commands

```bash
# Create
kubectl create -f deployment-definition.yaml

# List deployments
kubectl get deployments
kubectl get deploy        # shorthand

# See deployment + its ReplicaSet + its pods in one shot
kubectl get all

# Describe
kubectl describe deployment myapp-deployment

# Delete
kubectl delete deployment myapp-deployment

# Create imperatively (fastest in exam)
kubectl create deployment myapp --image=nginx --replicas=3
```

---

## Object Naming Convention

When a Deployment creates a ReplicaSet and pods, names are chained:

```
Deployment:   myapp-deployment
ReplicaSet:   myapp-deployment-6795844b58
Pod:          myapp-deployment-6795844b58-5rbjl
```

---

> [!tip] CKA Exam
> - In the exam, **always use Deployments** rather than bare ReplicaSets or Pods unless specifically told otherwise
> - `kubectl create deployment <name> --image=<image> --replicas=<n>` is the fastest imperative approach
> - `kubectl get all` is your best friend for a quick overview of what's running
> - Rolling updates and rollbacks are covered in depth in a later section — this note covers the basics
> - A Deployment automatically creates a ReplicaSet — you'll see both when running `kubectl get rs`
