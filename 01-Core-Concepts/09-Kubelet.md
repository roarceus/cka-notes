# Kubelet

## What It Does

The kubelet is the **node agent** — runs on every worker node and is the link between the control plane and the container runtime.

Responsibilities:
- Registers the node with the cluster
- Receives pod specs from the API server
- Instructs the container runtime to start/stop containers
- Monitors pod and container health
- Reports node and pod status back to the API server

---

## Key Distinction — kubeadm Does NOT Deploy Kubelet

Unlike every other control plane component, **kubeadm does not install the kubelet**. It must be manually installed on each worker node.

```bash
# Download and install manually
wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubelet

# Run as a service
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --register-node=true
```

---

## Inspecting the Kubelet

```bash
# Verify it's running and see active flags
ps -aux | grep kubelet

# Config file location (kubeadm clusters)
cat /var/lib/kubelet/config.yaml

# kubeconfig used by kubelet
cat /etc/kubernetes/kubelet.conf
```

---

> [!tip] CKA Exam
> - **kubeadm does not deploy the kubelet** — this is a common trick question
> - The kubelet is the only component that runs as a **system process**, not a pod
> - If a node shows `NotReady`, check the kubelet: `systemctl status kubelet` and `journalctl -u kubelet`
> - The kubelet config is at `/var/lib/kubelet/config.yaml` on kubeadm clusters
