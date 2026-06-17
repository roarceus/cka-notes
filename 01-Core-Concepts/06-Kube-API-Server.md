# Kube API Server

## What It Does

The kube-apiserver is the **front door to the cluster** — every operation goes through it. It is the only component that talks directly to etcd.

Responsibilities:
- Authenticates and validates all requests
- Reads/writes cluster state to etcd
- Coordinates between the scheduler, kubelet, and other components

---

## Request Lifecycle — Creating a Pod

![API Server request steps](https://kodekloud.com/kk-media/image/upload/v1752869720/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Kube-API-Server/frame_130.jpg)

1. Request arrives (via `kubectl` or direct API call)
2. API server **authenticates** the user
3. API server **validates** the request
4. Creates pod object in **etcd** (no node assigned yet)
5. **Scheduler** notices the unassigned pod → picks a node → tells API server
6. API server updates etcd with node assignment
7. API server notifies the **kubelet** on the target node
8. Kubelet instructs the container runtime to run the pod
9. Kubelet reports status back → API server updates etcd

---

## Inspecting the API Server

### kubeadm clusters
Runs as a static pod in `kube-system`:
```bash
kubectl get pods -n kube-system
# kube-apiserver-master   1/1   Running   0   15m

# View its config/flags
kubectl describe pod kube-apiserver-master -n kube-system

# Or read the manifest directly
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Manual (non-kubeadm) clusters
```bash
cat /etc/systemd/system/kube-apiserver.service
# or check the running process
ps -aux | grep kube-apiserver
```

---

## Key Configuration Flags

You don't need to memorise all flags, but know what these do:

| Flag | Purpose |
|---|---|
| `--etcd-servers` | Where etcd is (default: `https://127.0.0.1:2379`) |
| `--etcd-cafile` / `--etcd-certfile` / `--etcd-keyfile` | TLS certs for API server ↔ etcd |
| `--client-ca-file` | CA for authenticating clients (kubectl, etc.) |
| `--authorization-mode` | e.g. `Node,RBAC` |
| `--enable-admission-plugins` | e.g. `NodeRestriction`, `ResourceQuota` |
| `--service-cluster-ip-range` | IP range for ClusterIP services |
| `--advertise-address` | IP the API server advertises to the cluster |

---

> [!tip] CKA Exam
> - If something in the cluster is broken, **check the API server pod/logs first**: `kubectl describe pod kube-apiserver-master -n kube-system`
> - The static pod manifest lives at `/etc/kubernetes/manifests/kube-apiserver.yaml` — editing this file restarts the API server automatically
> - Know that `--authorization-mode=Node,RBAC` is the standard production setting
> - `--etcd-servers` tells you where the API server is looking for etcd — useful when diagnosing connectivity issues
