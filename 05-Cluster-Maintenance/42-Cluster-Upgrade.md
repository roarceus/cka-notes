# Cluster Upgrade Introduction

## Component Version Rules

No component should run on a version **higher** than the kube-apiserver.

| Component | Allowed versions relative to API Server |
|---|---|
| kube-apiserver | `X` (reference point) |
| controller-manager, kube-scheduler | `X` or `X-1` |
| kubelet, kube-proxy | `X`, `X-1`, or `X-2` |
| kubectl | `X+1`, `X`, or `X-1` (flexible) |

---

## When to Upgrade

Kubernetes officially supports the **3 most recent minor versions**. When a new version releases, the oldest drops out of support.

> Best practice: upgrade **one minor version at a time** (1.10 → 1.11 → 1.12), never skip versions.

---

## Upgrade Order

1. **Master nodes first** (control plane components)
   - Brief management outage during upgrade (kubectl, scaling temporarily unavailable)
   - Worker nodes keep running — pods continue serving traffic
   - Failed pods may not be restarted during this window

2. **Worker nodes** (one at a time — rolling approach)
   - Drain → upgrade → uncordon → repeat

---

## Worker Node Upgrade Strategies

| Strategy | Description | Downtime? |
|---|---|---|
| All at once | Upgrade all workers simultaneously | ✅ Yes |
| Rolling (one at a time) | Drain, upgrade, uncordon each node sequentially | ❌ No |
| Add new nodes | Provision new nodes at new version, migrate workloads, remove old | ❌ No |

---

## Key Facts About kubeadm Upgrades

- `kubeadm upgrade plan` — shows current versions, available upgrade targets
- `kubeadm upgrade apply <version>` — upgrades control plane components
- **kubeadm does NOT upgrade kubelet** — must be done manually on each node
- **Upgrade kubeadm itself first**, before running `kubeadm upgrade apply`
- `kubectl get nodes` shows the **kubelet version** on each node, not the API server version

---

> [!tip] CKA Exam
> - Always upgrade **one minor version at a time** — skipping versions is not supported
> - The upgrade order is always: **kubeadm → control plane → kubelet on each node**
> - `kubectl get nodes` showing old version after upgrade = kubelet not yet upgraded on that node
> - Worker node upgrade = `drain` → upgrade packages → `kubeadm upgrade node` → restart kubelet → `uncordon`
> - Detailed step-by-step commands are below

---

## Step-by-Step Upgrade (kubeadm — Ubuntu/Debian)

> Replace version numbers (e.g. `1.29.3-1.1`) with the target version. Always check `apt-cache madison kubeadm` for exact package names.

### 0. Update package repository to target version

```bash
# Point apt at the new minor version repo
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
```

---

### 1. Upgrade the Control Plane Node

```bash
# --- On the control plane node ---

# Upgrade kubeadm first
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm='1.29.3-1.1'
sudo apt-mark hold kubeadm

# Verify
kubeadm version

# Check what will be upgraded
sudo kubeadm upgrade plan

# Apply the upgrade
sudo kubeadm upgrade apply v1.29.3

# Drain control plane node
kubectl drain controlplane --ignore-daemonsets

# Upgrade kubelet and kubectl
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1'
sudo apt-mark hold kubelet kubectl

# Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# Uncordon
kubectl uncordon controlplane

# Verify
kubectl get nodes
```

---

### 2. Upgrade Each Worker Node

```bash
# --- Run on the worker node (e.g. node01) ---

# Upgrade kubeadm
sudo apt-mark unhold kubeadm
sudo apt-get install -y kubeadm='1.29.3-1.1'
sudo apt-mark hold kubeadm

# Update node config (run on the worker node itself)
sudo kubeadm upgrade node

# --- Back on control plane ---
kubectl drain node01 --ignore-daemonsets

# --- Back on the worker node ---
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet='1.29.3-1.1' kubectl='1.29.3-1.1'
sudo apt-mark hold kubelet kubectl

sudo systemctl daemon-reload
sudo systemctl restart kubelet

# --- Back on control plane ---
kubectl uncordon node01

# Verify all nodes
kubectl get nodes
```

---

### Key Differences: Control Plane vs Worker Node

| Step | Control Plane | Worker Node |
|---|---|---|
| kubeadm upgrade command | `kubeadm upgrade apply v1.29.x` | `kubeadm upgrade node` |
| Who drains it | Self (kubectl drain controlplane) | Control plane drains it |
| Order | First | After control plane |
