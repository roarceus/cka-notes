# Kube Proxy

## What It Does

kube-proxy runs on **every node** and maintains network rules that allow pods to communicate with Services. It watches for new Services and configures the node's networking accordingly.

---

## Why Services Need Kube Proxy

Pod IPs are **ephemeral** — they change when pods restart. A Service gives a stable IP/DNS name. kube-proxy makes that stable IP actually work by setting up **iptables rules** on each node:

```
Traffic to Service IP (10.96.0.12)
        ↓  [iptables rule set by kube-proxy]
Forwarded to Pod IP (10.32.0.15)
```

This redirection happens transparently on every node, so any pod on any node can reach the Service.

> A Service is a virtual construct — it's not a container or network interface. It only exists as rules in iptables (or ipvs).

---

## Deployment

kube-proxy is deployed as a **DaemonSet** by kubeadm — one pod per node, automatically.

```bash
# Verify kube-proxy is running
kubectl get pods -n kube-system | grep kube-proxy

# Check the DaemonSet
kubectl get daemonset -n kube-system
```

---

> [!tip] CKA Exam
> - kube-proxy is deployed as a **DaemonSet** — guaranteed one instance per node
> - It uses **iptables** (default) or **ipvs** to route Service traffic to pods
> - If a Service isn't reachable, kube-proxy and its iptables rules are worth investigating
> - Unlike the kubelet, kube-proxy **is** deployed automatically by kubeadm
