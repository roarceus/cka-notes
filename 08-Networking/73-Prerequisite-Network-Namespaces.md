# Prerequisite — Network Namespaces

Network namespaces are how containers get network isolation. Each pod in Kubernetes gets its own network namespace — understanding this explains how pod networking works.

---

## What Network Namespaces Provide

Each namespace has its own:
- Network interfaces
- Routing table
- ARP table

Processes inside the namespace only see their own network — completely isolated from the host and other namespaces.

---

## Key Commands

```bash
# Create a network namespace
ip netns add red
ip netns add blue

# List namespaces
ip netns

# Run a command inside a namespace
ip netns exec red ip link
ip -n red link          # shorthand

# View routing table inside namespace
ip netns exec red route

# View ARP table inside namespace
ip netns exec red arp
```

---

## Connecting Two Namespaces (veth pair)

A virtual ethernet (veth) pair is like a cable — one end in each namespace:

```bash
# Create veth pair
ip link add veth-red type veth peer name veth-blue

# Assign each end to its namespace
ip link set veth-red netns red
ip link set veth-blue netns blue

# Assign IPs
ip -n red addr add 192.168.15.1/24 dev veth-red
ip -n blue addr add 192.168.15.2/24 dev veth-blue

# Bring interfaces up
ip -n red link set veth-red up
ip -n blue link set veth-blue up

# Test connectivity
ip netns exec red ping 192.168.15.2
```

---

## Virtual Bridge (for multiple namespaces)

For many namespaces, use a Linux bridge (virtual switch) instead of direct veth pairs:

```bash
# Create bridge
ip link add v-net-0 type bridge
ip link set v-net-0 up

# Connect namespace to bridge via veth
ip link add veth-red type veth peer name veth-red-br
ip link set veth-red netns red
ip link set veth-red-br master v-net-0    # attach to bridge

# Assign IP to bridge (allows host to communicate with namespaces)
ip addr add 192.168.15.5/24 dev v-net-0
```

---

## How This Maps to Kubernetes

| Concept | Kubernetes equivalent |
|---|---|
| Network namespace | Each pod gets its own namespace |
| veth pair | Created by CNI plugin when pod starts |
| Bridge / virtual switch | CNI plugin creates a bridge on each node |
| NAT (iptables) | kube-proxy manages service iptables rules |

The CNI plugin (Calico, Flannel, etc.) automates all of this — you don't do it manually in Kubernetes.

---

> [!tip] CKA Exam
> - Every pod has its own network namespace — containers **within** the same pod share one namespace (that's how they communicate via `localhost`)
> - CNI plugins handle veth pair creation and bridge setup automatically
> - If debugging pod networking: `ip netns` on a node lists all pod network namespaces
> - `ip netns exec <ns> ip route` — check routes inside a pod's namespace for connectivity issues
