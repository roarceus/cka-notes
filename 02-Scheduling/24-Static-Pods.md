# Static Pods

## What They Are

Static pods are pods managed **directly by the kubelet** on a node — no API server, no scheduler, no etcd involved. The kubelet watches a local directory for pod manifests and creates/restarts/deletes pods based on what's there.

- Only **pod-level** resources can be static — no Deployments, ReplicaSets, or Services
- The kubelet automatically restarts static pods if they crash
- Used by kubeadm to run the control plane itself (apiserver, etcd, scheduler, controller-manager all run as static pods)

---

## Manifest Directory

The kubelet watches a directory for pod definition files. Configured via:

### Option 1 — direct flag
```bash
ExecStart=/usr/local/bin/kubelet \
  --pod-manifest-path=/etc/kubernetes/manifests \
  ...
```

### Option 2 — config file (kubeadm default)
```bash
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/config.yaml \
  ...
```
```yaml
# /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
```

**Default path on kubeadm clusters:** `/etc/kubernetes/manifests/`

---

## Finding the Static Pod Path on an Existing Node

```bash
# Check kubelet flags directly
ps -aux | grep kubelet | grep manifest

# Or find the config file and check staticPodPath
cat /var/lib/kubelet/config.yaml | grep staticPodPath
```

---

## Behaviour in a Cluster

When a node is part of a cluster, static pods also appear as **read-only mirror objects** in the API server:

```bash
kubectl get pods -A
# static pods show up with node name appended:
# kube-apiserver-controlplane   ← static pod mirror
# etcd-controlplane             ← static pod mirror
```

- You can **view** them with `kubectl` but **cannot delete or edit** them via kubectl
- To modify: edit the manifest file on the node directly → kubelet recreates the pod
- To delete: remove the manifest file from the directory

![Static pod architecture in a cluster](https://kodekloud.com/kk-media/image/upload/v1752869910/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Static-Pods/frame_340.jpg)

---

## Static Pods vs DaemonSets

| | Static Pods | DaemonSets |
|---|---|---|
| Managed by | kubelet directly | DaemonSet controller via API server |
| Needs API server | ❌ No | ✅ Yes |
| Scheduler involved | ❌ No | ❌ No (both bypass scheduler) |
| Scope | One specific node | All nodes |
| Typical use | Control plane components | Monitoring, logging, networking agents |

---

> [!tip] CKA Exam
> - Static pod manifests live at **`/etc/kubernetes/manifests/`** on kubeadm clusters
> - To create a static pod: drop a valid pod YAML into that directory — kubelet picks it up automatically
> - To delete a static pod: **remove the file** from the manifests directory — `kubectl delete` won't work
> - Static pod names always have the **node name appended** (e.g. `etcd-controlplane`) — easy to identify
> - If asked to troubleshoot a control plane component (etcd, apiserver, etc.), check `/etc/kubernetes/manifests/` — that's where their configs live
> - Find the manifest path: `ps -aux | grep kubelet` or `cat /var/lib/kubelet/config.yaml`
