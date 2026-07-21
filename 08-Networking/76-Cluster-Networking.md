# Cluster Networking

## Node Requirements

Every node must have:
- At least one network interface with an IP address
- A unique hostname
- A unique MAC address (important when cloning VMs)

---

## Required Ports

| Port | Component | Notes |
|---|---|---|
| **6443** | kube-apiserver | Must be open to all — workers, users, other control plane components |
| **10250** | kubelet | On all nodes (master + worker) |
| **10259** | kube-scheduler | Master only |
| **10257** | kube-controller-manager | Master only |
| **2379** | etcd | Client communication |
| **2380** | etcd | Peer-to-peer (HA multi-master only) |
| **30000–32767** | NodePort services | Worker nodes — external service exposure |

> These ports must be open in firewalls, iptables, and cloud security groups (AWS SGs, GCP firewall rules, Azure NSGs).

---

## Useful Verification Commands

```bash
# Network interfaces and IPs
ip link (ip link show eth0)
ip a
ip addr

# Routing table
ip route (ip route show default)

# IP forwarding status (must be 1 on all nodes)
cat /proc/sys/net/ipv4/ip_forward

# ARP table
arp

# Active listening ports and which process owns them
netstat -plnt
ss -tlnp          # modern alternative to netstat
netstat -anp
```

---

> [!tip] CKA Exam
> - **6443** is the API server port — the most important one to remember
> - **2379** = etcd client; **2380** = etcd peer (HA only)
> - **10250** = kubelet on every node
> - `netstat -plnt` or `ss -tlnp` to check which ports are listening — useful when troubleshooting component connectivity
> - If a node can't join the cluster, check firewall rules blocking 6443 or 10250
