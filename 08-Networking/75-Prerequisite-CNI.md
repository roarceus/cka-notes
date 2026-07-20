# Prerequisite — CNI (Container Network Interface)

## What CNI Is

CNI is a **standard interface** between container runtimes and network plugins. It defines how plugins should configure networking when containers start and stop — so any CNI plugin works with any CNI-compliant runtime.

---

## Why CNI Exists

Every container platform (Kubernetes, Rocket, Mesos) needs to:
- Create network namespaces
- Connect containers to networks via bridges/veth pairs
- Assign IP addresses
- Clean up when containers are deleted

Rather than each platform implementing this differently, CNI standardises it with a plugin model.

---

## How It Works

When Kubernetes creates a pod, it:
1. Creates a network namespace for the pod
2. Calls the CNI plugin's **`add`** command with the container ID and namespace
3. The plugin sets up interfaces, assigns IPs, configures routes

When the pod is deleted, Kubernetes calls the **`del`** command to clean up.

```bash
# CNI plugin interface (what plugins must implement)
bridge add <container-id> /var/run/netns/<ns>
bridge del <container-id> /var/run/netns/<ns>
```

---

## Docker vs CNI

> Docker uses its own standard called **CNM (Container Network Model)** — not CNI compatible.

Kubernetes works around this by creating Docker containers with `--network=none` and then calling the CNI plugin separately to set up networking.

---

## Common CNI Plugins

| Plugin | Type |
|---|---|
| `bridge`, `vlan`, `ipvlan`, `macvlan` | Built-in |
| Weave, Flannel, Cilium, Calico | Third-party overlay networks |
| `host-local`, `dhcp` | IPAM (IP address management) |

---

> [!tip] CKA Exam
> - CNI plugins are configured in `/etc/cni/net.d/` on each node
> - CNI binaries live in `/opt/cni/bin/`
> - The CNI plugin in use determines how pods get IPs and how cross-node pod routing works
> - `kubectl get pods -n kube-system` — your CNI plugin runs here (e.g. `weave-net`, `calico-node`, `cilium`)
> - If pods are stuck `Pending` or can't communicate, check the CNI plugin pods and config
