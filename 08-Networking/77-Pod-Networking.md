# Pod Networking

## Kubernetes Pod Networking Requirements

Kubernetes mandates that any networking solution must satisfy these rules:
- Every pod gets a **unique IP address**
- Pods on the **same node** can reach each other via IP
- Pods on **different nodes** can reach each other via IP — **without NAT**

---

## How It Works Under the Hood

Each node runs a **bridge network** that connects pods on that node. The CNI plugin automates this entirely.

### Per-node setup (done by CNI plugin automatically)

```
Node 1 (192.168.1.11):  bridge 10.244.1.0/24
Node 2 (192.168.1.12):  bridge 10.244.2.0/24
Node 3 (192.168.1.13):  bridge 10.244.3.0/24
```

When a pod is created:
1. CNI creates a network namespace for the pod
2. Creates a veth pair — one end in the pod namespace, one end on the bridge
3. Assigns an IP from the node's pod CIDR (e.g. `10.244.1.2`)
4. Adds a default route inside the pod namespace

### Cross-node routing

For pods on different nodes to communicate, each node needs routes to the other nodes' pod subnets:

```bash
# On node 1 — route to node 2's pods
ip route add 10.244.2.0/24 via 192.168.1.12

# On node 1 — route to node 3's pods
ip route add 10.244.3.0/24 via 192.168.1.13
```

CNI plugins (Calico, Flannel, Weave, etc.) manage this automatically — either via host routes, BGP, overlay networks (VXLAN), or a central router.

---

## Pod CIDR

Each node is assigned a slice of the cluster's pod CIDR. Check node pod CIDRs:

```bash
kubectl get nodes -o wide
kubectl describe node node01 | grep PodCIDR
```

---

## CNI Role in Pod Networking

The kubelet calls the CNI plugin's `add` command each time a pod is created:

```bash
./net-script.sh add <container-id> <namespace>
```

The plugin handles everything: veth pair, IP assignment, routes. On pod deletion, the `del` command cleans up.

CNI config files live at: `/etc/cni/net.d/`
CNI plugin binaries at: `/opt/cni/bin/`

---

> [!tip] CKA Exam
> - Pod IPs come from the **pod CIDR** assigned to each node — check it with `kubectl describe node`
> - All pod-to-pod communication must work without NAT — that's the CNI plugin's job
> - If pods can't reach pods on other nodes: check node routes (`ip route`) and CNI plugin pods
> - The cluster pod CIDR is configured at cluster creation (e.g. `--pod-network-cidr=10.244.0.0/16` in kubeadm)
> - `kubectl get pods -n kube-system` — your CNI plugin pods run here
