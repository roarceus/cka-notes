# CNI in Kubernetes

## How Kubernetes Uses CNI

When a pod is created, the container runtime (containerd, CRI-O) invokes the CNI plugin to set up networking. The kubelet tells the runtime which CNI config to use.

---

## Key Directories

```bash
# CNI plugin binaries — executables the runtime calls
ls /opt/cni/bin
# bridge  dhcp  flannel  host-local  ipvlan  loopback  macvlan  weave-net...

# CNI network config files — defines which plugin and settings to use
ls /etc/cni/net.d
# 10-bridge.conflist  (runtime picks the first alphabetically)
```

---

## CNI Config File Explained

```json
{
  "cniVersion": "0.2.0",
  "name": "mynet",
  "type": "bridge",         // plugin binary to call from /opt/cni/bin/
  "bridge": "cni0",         // name of the bridge interface on the node
  "isGateway": true,        // bridge gets an IP address, acts as gateway for pods
  "ipMasq": true,           // enable NAT/masquerade for outbound traffic
  "ipam": {
    "type": "host-local",   // IP address management plugin
    "subnet": "10.22.0.0/16",
    "routes": [
      { "dst": "0.0.0.0/0" }   // default route for pods
    ]
  }
}
```

---

## Checking the CNI Plugin in Use

```bash
# See what CNI configs are installed
ls /etc/cni/net.d/

# See what plugin binaries are available
ls /opt/cni/bin/

# Check which CNI plugin pods are running
kubectl get pods -n kube-system | grep -E "calico|flannel|weave|cilium|canal"
```

---

> [!tip] CKA Exam
> - Runtime picks the **first config file alphabetically** from `/etc/cni/net.d/`
> - Plugin binaries must exist in `/opt/cni/bin/` — if missing, pods will fail to start with a network error
> - If pods are stuck in `ContainerCreating`, CNI misconfiguration is a common cause — check `/etc/cni/net.d/` and CNI plugin pod logs
> - `"ipam"` section handles IP address assignment — `host-local` allocates from a local subnet, `dhcp` uses DHCP
