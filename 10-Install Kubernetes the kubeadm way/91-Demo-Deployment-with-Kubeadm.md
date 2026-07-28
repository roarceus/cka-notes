# Demo Deployment with Kubeadm

## 1. Identify Node IPs

Before initializing anything, check each node's network interfaces to find the IP to use for cluster communication:

```bash
ip add
```

- Look for the static/internal IP (e.g. the `192.168.x.x` interface) — this is what you'll use for `--apiserver-advertise-address` and worker join commands, not a dynamic NAT IP like `10.0.2.15`

---

## 2. Prerequisites (all nodes)

- Supported Linux distro (e.g. Ubuntu)
- Minimum **2 GB RAM** and **2 CPUs** per node
- Required kernel modules loaded (br_netfilter, overlay, etc.)
- Relevant sysctl network variables set to `1` so bridged traffic is visible to iptables

---

## 3. Install Container Runtime — containerd (all nodes)

Add the Kubernetes apt repo:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core/stable/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes.gpg] https://pkgs.k8s.io/core/stable/v1.31/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

Install containerd:
```bash
sudo apt-get install -y containerd
sudo systemctl status containerd
```

**Critical**: containerd and kubelet must use the **same cgroup driver**. If systemd is the init system (`ps -p 1` confirms this), configure containerd to use it:
```bash
sudo mkdir -p /etc/containerd
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

> [!tip] CKA Exam
> Mismatched cgroup drivers between containerd and kubelet is a classic source of kubelet startup failures. If a node won't join or kubelet won't start, check this first.

---

## 4. Install kubeadm, kubelet, kubectl (all nodes)

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable --now kubelet
```

- `apt-mark hold` prevents these from being auto-upgraded (Kubernetes versions must be upgraded deliberately/carefully)
- **kubeadm**: bootstraps and manages cluster init/join
- **kubelet**: manages pods/containers on each node
- **kubectl**: CLI to interact with the cluster

---

## 5. Initialize the Control Plane (master only)

```bash
sudo kubeadm init --apiserver-advertise-address=192.168.56.11 --pod-network-cidr="10.244.0.0/16" --upload-certs
```

- `--apiserver-advertise-address`: the master's static IP (from step 1)
- `--pod-network-cidr`: must match what your chosen CNI plugin expects (e.g. `10.244.0.0/16` for Flannel/Weave)
- `--upload-certs`: uploads control-plane certs so additional masters can join later (HA setups)

Configure kubectl access as a regular user:
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify:
```bash
kubectl get pods -A
```
- No pods stuck in `Pending`/errors in the core namespaces confirms basic init success (though CoreDNS pods will stay `Pending` until the pod network is installed)

---

## 6. Deploy a Pod Network Add-on (master only)

Pods can't communicate cluster-wide until a CNI plugin is installed:
```bash
kubectl apply -f [podnetwork].yaml
```
- Replace with the actual manifest URL for your chosen CNI (e.g. Weave Net, Flannel, Calico)
- Deploys as a DaemonSet — runs on the control plane first, then propagates once workers join

Verify:
```bash
kubectl get pods -A
```
- CNI pods (e.g. `weave-net-*`) should move to `Running`, and CoreDNS pods should leave `Pending`

---

## 7. Join Worker Nodes

Run the exact `kubeadm join` command output at the end of `kubeadm init` (on each worker node):

```bash
sudo kubeadm join 192.168.56.11:6443 --token ps4rl5.0ns9vwu9exjul8tg \
    --discovery-token-ca-cert-hash sha256:fdb5c133b76f41d6d1f9ed72d90b7265de5e53a9156d7d48d83df65f3bde
```

Verify from the master:
```bash
kubectl get nodes
```
- All nodes (master + workers) should show `Ready`

> [!tip] CKA Exam
> If you lose the join command/token, regenerate it on the master with `kubeadm token create --print-join-command`.

---

## 8. Verify the Cluster

```bash
kubectl run nginx --image=nginx
kubectl get pod
```
- Confirms scheduling, pulling, and running a pod actually works end-to-end
- Delete the test pod once verified

---

> [!tip] CKA Exam
> - Order: container runtime → kubeadm/kubelet/kubectl → `kubeadm init` (master) → pod network add-on → `kubeadm join` (workers)
> - `--pod-network-cidr` on `kubeadm init` must match the CNI plugin's expected range
> - Nodes won't show `Ready` and CoreDNS stays `Pending` until a pod network add-on is applied
> - `apt-mark hold` on kubelet/kubeadm/kubectl prevents accidental version drift
