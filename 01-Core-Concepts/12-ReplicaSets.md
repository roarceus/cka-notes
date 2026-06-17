# ReplicaSets

## What They Do

ReplicaSets (and the older ReplicationController) ensure a **specified number of pod replicas are always running**. If a pod crashes, the ReplicaSet automatically creates a replacement.

Two purposes:
- **High availability** — replace failed pods automatically
- **Load balancing / scaling** — spread pods across nodes as demand grows

---

## ReplicationController vs ReplicaSet

| | ReplicationController | ReplicaSet |
|---|---|---|
| API version | `v1` | `apps/v1` |
| Selector | Implicit (matches template labels) | Explicit (`selector.matchLabels`) — **required** |
| Status | Legacy, being phased out | ✅ Current standard |

> Always use **ReplicaSet** — ReplicationController is legacy.

---

## ReplicaSet YAML

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end        # must match labels in template below
  template:                  # pod blueprint — required even if pods already exist
    metadata:
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
```

Key points:
- `selector.matchLabels` must match the labels in `template.metadata.labels`
- The `template` section is always required — it's the blueprint for replacement pods
- Pod names will be prefixed with the ReplicaSet name (e.g. `myapp-replicaset-4lvk9`)

---

## Labels & Selectors

Labels let the ReplicaSet identify which pods it manages. A ReplicaSet can even **adopt pre-existing pods** if their labels match the selector.

```yaml
selector:
  matchLabels:
    tier: front-end   # targets any pod with this label
```

---

## Essential Commands

```bash
# Create
kubectl create -f replicaset-definition.yaml

# List
kubectl get replicaset
kubectl get rs           # shorthand

# Describe
kubectl describe replicaset myapp-replicaset

# Delete (also deletes all managed pods)
kubectl delete replicaset myapp-replicaset

# Edit live object
kubectl edit replicaset myapp-replicaset
```

---

## Scaling

```bash
# Method 1: update replicas in YAML, then apply
kubectl replace -f replicaset-definition.yaml

# Method 2: scale directly (does NOT update the file)
kubectl scale --replicas=6 -f replicaset-definition.yaml
kubectl scale --replicas=6 replicaset myapp-replicaset
```

---

> [!tip] CKA Exam
> - ReplicaSet `apiVersion` is **`apps/v1`** — forgetting this is a common YAML error
> - The `selector` field is **mandatory** in a ReplicaSet (unlike ReplicationController)
> - `selector.matchLabels` **must match** `template.metadata.labels` exactly
> - `kubectl scale replicaset <name> --replicas=<n>` is the fastest way to scale in the exam
> - Deleting a ReplicaSet deletes all its pods — use `kubectl delete rs <name>`
> - To scale without deleting/recreating: `kubectl edit rs <name>` and change `replicas`
