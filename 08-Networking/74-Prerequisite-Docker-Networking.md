# Prerequisite — Docker Networking

Docker networking is the foundation for understanding how Kubernetes pod networking works — CNI plugins replicate the same concepts automatically.

---

## Docker Network Modes

| Mode | Command | Behaviour |
|---|---|---|
| `none` | `--network none` | No network — fully isolated |
| `host` | `--network host` | Shares host's network stack — no isolation, no port mapping needed |
| `bridge` | *(default)* | Private internal network via `docker0` bridge — each container gets its own IP |

```bash
docker run --network none nginx
docker run --network host nginx
docker run nginx                  # uses bridge by default
```

---

## Bridge Network Internals

When Docker creates a container on the bridge network it:
1. Creates a new **network namespace** for the container
2. Creates a **veth pair** (virtual cable)
3. Places one end in the container namespace (`eth0`), attaches the other to `docker0` bridge
4. Assigns the container an IP from `172.17.0.0/16`

```bash
# View Docker networks
docker network ls

# Inspect host-side interfaces (veth pairs)
ip link show

# View docker0 bridge
ip link show docker0
# IP: 172.17.0.1
```

---

## Port Mapping

Containers on the bridge network are only reachable from the host by default. Port mapping exposes them externally:

```bash
# Map host port 8080 → container port 80
docker run -p 8080:80 nginx

# Access via host IP
curl http://192.168.1.10:8080    # works
curl http://172.17.0.3:80        # only works from the host
```

**How it works:** Docker adds an iptables NAT rule:
```bash
# Docker creates a rule like:
iptables -t nat -A PREROUTING -j DNAT --dport 8080 --to-destination 172.17.0.2:80

# Inspect active rules
iptables -nvL -t nat
```

---

## How This Maps to Kubernetes

| Docker | Kubernetes equivalent |
|---|---|
| `docker0` bridge | Node-level bridge created by CNI plugin |
| veth pair per container | veth pair per pod (created by CNI) |
| Port mapping (`-p`) | NodePort Service / kube-proxy iptables rules |
| Container network namespace | Pod network namespace |

---

> [!tip] CKA Exam
> - Pods use a bridge-like setup managed by the CNI plugin — same concept as `docker0` but at the cluster level
> - kube-proxy manages iptables rules for Service routing — same mechanism as Docker port mapping
> - `host` network mode pods share the node's network stack — useful for system-level workloads but breaks pod network isolation
