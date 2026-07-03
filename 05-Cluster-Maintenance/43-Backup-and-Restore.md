# Backup and Restore Methods

## What to Back Up

| What | Why | How |
|---|---|---|
| **Declarative YAML files** | Best — store in git, reusable | Version control (GitHub etc.) |
| **All cluster resources** | Captures imperative objects too | `kubectl get all --all-namespaces -o yaml` |
| **etcd snapshot** | Full cluster state — the most complete backup | `etcdctl snapshot save` |

---

## Method 1 — Backup All Resources via API Server

```bash
# Dump everything across all namespaces to a file
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

> Best for managed clusters where you don't have direct etcd access. Tools like **Velero** automate this in production.

---

## etcdctl vs etcdutl

Two separate tools — easy to mix up:

| Tool | Used for | Requires running etcd? |
|---|---|---|
| `etcdctl` | Snapshot save, snapshot status | ✅ Yes (connects to live etcd) |
| `etcdutl` | Snapshot restore, file-level backup | ❌ No (works offline) |

### Verify etcdctl API version
```bash
etcdctl version
# etcdctl version: 3.5.16
# API version: 3.5
```

---

## Method 2 — etcd Snapshot (most complete)

### Take a snapshot

```bash
# Minimal (local etcd)
ETCDCTL_API=3 etcdctl snapshot save snapshot.db

# With authentication (required on most clusters)
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

### Verify the snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
# or
ETCDCTL_API=3 etcdctl snapshot status snapshot.db --write-out=table
```

---

## Restore from etcd Snapshot

```bash
# Step 1 — Stop the API server (on non-kubeadm setups)
service kube-apiserver stop

# Step 2 — Restore snapshot using etcdutl (not etcdctl)
etcdutl snapshot restore snapshot.db \
  --data-dir=/var/lib/etcd-from-backup

# Step 3 — Update etcd to point to the new data directory
# Edit /etc/kubernetes/manifests/etcd.yaml:
#   --data-dir=/var/lib/etcd-from-backup
# Also update the hostPath volume to match

# Step 4 — Reload and restart services
systemctl daemon-reload
systemctl restart etcd
service kube-apiserver start   # non-kubeadm only
```

> On kubeadm clusters, etcd is a static pod — editing the manifest at `/etc/kubernetes/manifests/etcd.yaml` automatically restarts it.

---

## Alternative: File-level Backup with etcdutl

Works offline — copies the raw etcd data and WAL files without connecting to a running etcd:

```bash
# Backup
etcdutl backup \
  --data-dir /var/lib/etcd \
  --backup-dir /backup/etcd-backup

# Restore: just copy the backup contents back to /var/lib/etcd and restart etcd
```

---

## Finding etcd Certificate Paths

```bash
# Check the etcd static pod for cert paths
cat /etc/kubernetes/manifests/etcd.yaml | grep -E "cert|key|ca"

# Or describe the etcd pod
kubectl describe pod etcd-controlplane -n kube-system
```

Common paths on kubeadm clusters:
```
--cacert=/etc/kubernetes/pki/etcd/ca.crt
--cert=/etc/kubernetes/pki/etcd/server.crt
--key=/etc/kubernetes/pki/etcd/server.key
```

---

> [!tip] CKA Exam
> - Always set `export ETCDCTL_API=3` before running `etcdctl` commands
> - **`etcdctl`** = save + status (needs live etcd); **`etcdutl`** = restore + file backup (offline)
> - The cert flags are almost always required — find paths from the etcd pod manifest
> - `snapshot save` → `etcdctl`; `snapshot restore` → `etcdutl` (common exam gotcha)
> - After restore, update the `--data-dir` flag **and** the corresponding `hostPath` volume in the etcd manifest
> - `kubectl get all` doesn't capture everything — CustomResourceDefinitions, PersistentVolumes, etc. are missed; etcd snapshot is the only truly complete backup
> - For managed clusters (EKS, GKE, AKS), use `kubectl get all -o yaml` — etcd is not directly accessible
