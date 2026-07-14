# Persistent Volumes (PVs)

## Why PVs Exist

Instead of embedding storage config in every pod definition, admins create a **centralised pool of storage** (PVs). Users then request slices of that pool via **Persistent Volume Claims (PVCs)**.

```
Admin creates PV  →  User creates PVC  →  PVC binds to PV  →  Pod uses PVC
```

---

## Access Modes

| Mode | Meaning |
|---|---|
| `ReadWriteOnce` (RWO) | Read-write by **one node** at a time |
| `ReadOnlyMany` (ROX) | Read-only by **many nodes** |
| `ReadWriteMany` (RWX) | Read-write by **many nodes** |
| `ReadWriteOncePod` (RWOP) | Read-write by **one pod** (Kubernetes 1.22+) |

---

## PV YAML

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  persistentVolumeReclaimPolicy: Retain    # what happens when PVC is deleted
  hostPath:                                # storage backend
    path: /tmp/data
```

### With AWS EBS (production example)
```yaml
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

---

## Reclaim Policies

What happens to the PV when the PVC that was bound to it is deleted:

| Policy | Behaviour |
|---|---|
| `Retain` | PV stays — data preserved, must be manually reclaimed |
| `Delete` | PV and underlying storage are deleted automatically |
| `Recycle` | Data is scrubbed, PV made available again (deprecated) |

---

## Commands

```bash
kubectl create -f pv.yaml

kubectl get persistentvolume
kubectl get pv                     # shorthand

kubectl describe pv pv-vol1
```

---

> [!tip] CKA Exam
> - PVs are **cluster-scoped** — no namespace
> - `hostPath` is for single-node testing only — use cloud/NFS backends in production
> - A PV stays in `Available` state until a PVC binds to it
> - After a PVC is deleted, PV status depends on `persistentVolumeReclaimPolicy` — `Retain` leaves it in `Released` state (not automatically reusable)
> - PVs are created by admins; PVCs are created by users/developers — know who does what
