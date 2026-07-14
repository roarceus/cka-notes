# Persistent Volume Claims (PVCs)

## What They Do

A PVC is a **user's request for storage**. Kubernetes automatically finds a matching PV and binds them together.

```
PVC created → Kubernetes finds matching PV → PVC status: Bound → Pod uses PVC
```

---

## PVC YAML

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
  storageClassName: ""        # optional: "" means use a static PV (no dynamic provisioning)
  selector:                   # optional: bind to a specific PV by label
    matchLabels:
      release: "stable"
```

---

## Binding Rules

Kubernetes matches a PVC to a PV based on:
- **Access modes** must be compatible
- **Capacity** — PV must have at least as much as PVC requests
- **Storage class** must match
- **Selectors** (if specified)

> If a 500Mi PVC binds to a 1Gi PV, the remaining 500Mi is **unavailable to any other PVC** — a PV can only bind to one PVC.

If no matching PV exists, PVC stays in `Pending` until one becomes available.

---

## Using a PVC in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - mountPath: /var/www/html
      name: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: myclaim        # reference the PVC name
```

---

## Commands

```bash
kubectl create -f pvc.yaml

kubectl get persistentvolumeclaim
kubectl get pvc                   # shorthand

kubectl describe pvc myclaim      # check status and bound PV

kubectl delete pvc myclaim
```

---

## PVC Lifecycle States

| State | Meaning |
|---|---|
| `Pending` | No matching PV found yet |
| `Bound` | Successfully matched and bound to a PV |
| `Lost` | The bound PV is no longer available |

---

> [!tip] CKA Exam
> - PVCs are **namespace-scoped**; PVs are cluster-scoped
> - A PV can only be bound to **one PVC at a time** — 1:1 relationship
> - After deleting a PVC, the PV's fate depends on `persistentVolumeReclaimPolicy` (see PV note)
> - `Retain` → PV becomes `Released` — **not** automatically `Available` for new PVCs
> - To use a PVC in a pod: add it under `volumes` with `persistentVolumeClaim.claimName` then mount via `volumeMounts`
> - If a PVC is stuck `Pending`, check: is there a PV with matching access mode, capacity, and storage class?
