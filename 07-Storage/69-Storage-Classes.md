# Storage Classes

## Static vs Dynamic Provisioning

| | Static | Dynamic |
|---|---|---|
| Who creates the PV? | Admin manually | Kubernetes automatically |
| Requires pre-created disk? | ✅ Yes | ❌ No |
| How? | Create PV → PVC binds to it | Create StorageClass → PVC references it → PV auto-created |

---

## StorageClass YAML

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd    # the storage backend plugin
parameters:                           # optional — provisioner-specific
  type: pd-ssd
  replication-type: none
```

---

## Dynamic Provisioning Flow

1. Admin creates a StorageClass
2. User creates a PVC referencing the StorageClass
3. Kubernetes calls the provisioner → creates the disk + PV automatically
4. PVC binds to the auto-created PV
5. Pod uses the PVC

```yaml
# PVC referencing a StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: google-storage    # must match StorageClass name
  resources:
    requests:
      storage: 500Mi
```

---

## Multiple Tiers (Silver / Gold / Platinum)

```yaml
# Silver — standard disk
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: silver
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  replication-type: none
---
# Gold — SSD
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: none
---
# Platinum — SSD with regional replication
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: platinum
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd
```

---

## Common Provisioners

| Provisioner | Backend |
|---|---|
| `kubernetes.io/gce-pd` | GCP Persistent Disk |
| `kubernetes.io/aws-ebs` | AWS EBS |
| `kubernetes.io/azure-disk` | Azure Disk |
| `kubernetes.io/no-provisioner` | Local storage (no dynamic provisioning) |
| `docker.io/portworx-volume` | Portworx |

---

## Commands

```bash
kubectl get storageclass
kubectl get sc              # shorthand

kubectl describe sc google-storage
```

---

> [!tip] CKA Exam
> - StorageClass `apiVersion` is **`storage.k8s.io/v1`**
> - When a PVC specifies `storageClassName`, a PV is **automatically created** — no manual PV needed
> - `storageClassName: ""` in a PVC means use a pre-existing static PV (no dynamic provisioning)
> - The **default** StorageClass is used when a PVC doesn't specify `storageClassName` — check with `kubectl get sc` (marked with `(default)`)
> - `kubectl get pvc` showing `Pending` after referencing a StorageClass = check provisioner is installed and parameters are correct
