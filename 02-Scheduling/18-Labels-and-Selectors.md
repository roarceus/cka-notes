# Labels and Selectors

## What They Are

- **Labels** — key/value pairs attached to objects for identification and grouping
- **Selectors** — queries that filter objects by their labels
- **Annotations** — metadata attached to objects but NOT used for selection (e.g. build version, contact info)

---

## Labels on a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    app: App1           # label key: value
    function: Front-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
```

```bash
# Filter pods by label
kubectl get pods --selector app=App1
kubectl get pods -l app=App1          # shorthand

# Multiple selectors
kubectl get pods -l app=App1,function=Front-end

# Works on any resource type
kubectl get all --selector app=App1
```

---

## Labels in ReplicaSets (two places)

Labels appear in **two distinct places** in a ReplicaSet — don't confuse them:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:              # 1. Labels ON the ReplicaSet itself
    app: App1
  annotations:
    buildversion: "1.34"   # metadata only — not used for selection
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1        # 2. Selector — must match labels in template below
  template:
    metadata:
      labels:
        app: App1      # 3. Labels ON the pods (must match selector above)
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
```

| Label location | Purpose |
|---|---|
| `metadata.labels` (ReplicaSet) | Lets other objects select *this* ReplicaSet |
| `spec.selector.matchLabels` | Tells the ReplicaSet which pods to manage |
| `spec.template.metadata.labels` | Labels applied to each pod — must match selector |

---

## Annotations

Used to store non-identifying metadata — **not** used for filtering:

```yaml
metadata:
  annotations:
    buildversion: "1.34"
    contact: "team@example.com"
```

---

> [!tip] CKA Exam
> - `kubectl get pods -l key=value` is the fastest way to filter by label
> - The `selector.matchLabels` in a ReplicaSet/Deployment **must exactly match** `template.metadata.labels` — mismatch causes an error
> - Services use the same label selector mechanism to target pods
> - `kubectl get all -l app=App1` finds every resource type with that label at once — useful for debugging
