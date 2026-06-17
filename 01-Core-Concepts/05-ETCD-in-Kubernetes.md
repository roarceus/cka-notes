# ETCD in Kubernetes

## Role of etcd in Kubernetes

etcd is the **single source of truth** for the entire cluster. It stores:
- Nodes, Pods, ConfigMaps, Secrets
- Accounts, Roles, RoleBindings
- ReplicaSets, Deployments — every Kubernetes object

> A change is only considered **complete** once it's been written to etcd. `kubectl get` reads from etcd.

---

## Deployment Methods

### 1. Manual (from scratch)
You download, install, and configure etcd as a systemd service on the master node. Gives full control over TLS and clustering options.

Key flags to know:
```bash
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://${INTERNAL_IP}:2379 \        # <-- API server uses this
  --listen-peer-urls https://${INTERNAL_IP}:2380 \             # <-- peer-to-peer (HA)
  --initial-cluster controller-0=https://${CONTROLLER0_IP}:2380,controller-1=https://${CONTROLLER1_IP}:2380 \
  --data-dir=/var/lib/etcd
```

Key ports:
- **2379** — client requests (API server talks to etcd here)
- **2380** — peer-to-peer communication between etcd members (HA only)

### 2. kubeadm
etcd is deployed automatically as a **static pod** in the `kube-system` namespace.

```bash
kubectl get pods -n kube-system
# etcd-master   1/1   Running   0   1h
```

---

## Inspecting etcd Data

All Kubernetes objects are stored under `/registry/` in etcd:

```bash
# List all keys stored in etcd
kubectl exec etcd-master -n kube-system -- \
  etcdctl get / --prefix --keys-only
```

Sample output:
```
/registry/pods/default/my-pod
/registry/deployments/default/my-deployment
/registry/secrets/default/my-secret
```

---

## High Availability

In HA setups, multiple etcd instances run across master nodes. Each instance must know about its peers via `--initial-cluster`:

```bash
--initial-cluster \
  controller-0=https://${CONTROLLER0_IP}:2380,\
  controller-1=https://${CONTROLLER1_IP}:2380
```

Uses the **Raft consensus protocol** to elect a leader and keep data consistent across members. HA etcd covered in depth in the HA section of the course.

---

> [!tip] CKA Exam
> - Know the difference between port **2379** (clients) and **2380** (peers)
> - `--advertise-client-urls` is what the API server uses to reach etcd — check this if the API server can't connect
> - With kubeadm, etcd is a pod: `kubectl exec etcd-master -n kube-system -- etcdctl ...`
> - Always set `export ETCDCTL_API=3` before running `etcdctl` commands in the exam
> - For kubeadm clusters, you'll also need to pass TLS certs to `etcdctl`:
> ```bash
> kubectl exec etcd-master -n kube-system -- etcdctl \
>   --cacert /etc/kubernetes/pki/etcd/ca.crt \
>   --cert /etc/kubernetes/pki/etcd/server.crt \
>   --key /etc/kubernetes/pki/etcd/server.key \
>   get / --prefix --keys-only
> ```
