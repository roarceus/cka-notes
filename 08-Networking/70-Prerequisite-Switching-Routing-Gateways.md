# Prerequisite — Switching, Routing & Gateways

Essential Linux networking commands needed for Kubernetes node and cluster troubleshooting.

---

## Key Commands

```bash
# List network interfaces and their state
ip link

# View assigned IP addresses
ip addr
ip addr show eth0

# Assign an IP address (temporary)
ip addr add 192.168.1.10/24 dev eth0

# View routing table
route
# or
ip route
ip route show

# Add a route to a specific network
ip route add 192.168.2.0/24 via 192.168.1.1

# Add a default gateway (for internet / unknown destinations)
ip route add default via 192.168.2.1

# Check IP forwarding status (must be 1 on nodes acting as routers)
cat /proc/sys/net/ipv4/ip_forward

# Enable IP forwarding temporarily
echo 1 > /proc/sys/net/ipv4/ip_forward

# Enable IP forwarding permanently
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

---

## Quick Reference

| Concept | Command |
|---|---|
| List interfaces | `ip link` |
| View IPs | `ip addr` |
| Assign IP | `ip addr add <ip/prefix> dev <iface>` |
| View routes | `ip route` |
| Add route | `ip route add <network> via <gateway>` |
| Default gateway | `ip route add default via <gateway>` |
| Check forwarding | `cat /proc/sys/net/ipv4/ip_forward` |

---

## IP Forwarding — Why It Matters for Kubernetes

Kubernetes nodes act as routers for pod traffic. IP forwarding **must be enabled** on all nodes:

```bash
# Check
cat /proc/sys/net/ipv4/ip_forward
# Must return: 1

# Make permanent (required for Kubernetes nodes)
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

> ⚠️ Changes with `ip` commands are **temporary** (lost on reboot) unless written to config files.

---

> [!tip] CKA Exam
> - `ip link` / `ip addr` / `ip route` are the go-to commands for diagnosing network issues on nodes
> - If pods can't communicate across nodes, check routes and IP forwarding on the nodes
> - `ip route show` to verify pod CIDR routes are present on each node
> - These commands are used in node-level networking troubleshooting tasks
