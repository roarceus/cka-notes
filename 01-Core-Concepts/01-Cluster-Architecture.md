# Cluster Architecture

## The Two-Node Model

Kubernetes clusters are made of two types of nodes:

- **Master Node (Control Plane)** — the brain; manages the cluster, makes scheduling decisions, stores state
- **Worker Nodes** — the muscle; run the actual containerized workloads

![Kubernetes Master and Worker Nodes](https://kodekloud.com/kk-media/image/upload/v1752869701/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Cluster-Architecture/frame_140.jpg)

---

## Master Node Components

![Master Node Architecture](https://kodekloud.com/kk-media/image/upload/v1752869703/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Cluster-Architecture/frame_270.jpg)

### etcd
A distributed key-value store that holds **all cluster state and configuration** — what nodes exist, what pods are running, what the desired state is. Uses a quorum mechanism for consistency.

### kube-scheduler
Watches for newly created pods with no assigned node and **selects the best node** to run them on, based on:
- Available resources
- Taints & tolerations
- Node affinity rules

### Controllers
Run control loops to **reconcile actual state with desired state**. Key controllers include:
- **Node Controller** — monitors node health
- **Replication Controller** — ensures correct pod counts are running

### kube-apiserver
The **central communication hub** of the cluster. All components (including external tools like `kubectl`) talk to each other through it. It's the only component that talks directly to etcd.

---

## Worker Node Components

![Worker Node Architecture](https://kodekloud.com/kk-media/image/upload/v1752869704/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Cluster-Architecture/frame_510.jpg)

### kubelet
The **node agent** — runs on every worker node. It:
- Registers the node with the cluster
- Receives pod specs from the API server
- Starts, stops, and monitors containers
- Reports node and pod status back to the API server

### kube-proxy
Manages **networking rules** on each node so that pods can communicate across nodes. For example, it enables a web pod on Node A to reach a database pod on Node B.

### Container Runtime
Every node needs a container runtime to actually run containers. Supported runtimes (via CRI):
- `containerd` *(most common today)*
- `CRI-O`
- `Docker` *(legacy)*

---

## Quick Reference Table

| Component | Lives On | Role |
|---|---|---|
| etcd | Master | Cluster state store |
| kube-apiserver | Master | Central API hub |
| kube-scheduler | Master | Assigns pods to nodes |
| Controllers | Master | Maintain desired state |
| kubelet | Worker | Node agent, runs pods |
| kube-proxy | Worker | Pod networking rules |
| Container Runtime | All nodes | Runs containers |

---

> [!tip] CKA Exam
> - Know **what each component does** and **where it runs** (master vs worker)
> - The **kube-apiserver** is the only component that reads/writes to etcd directly
> - The **kubelet** is the only component that runs as a system process (not a pod) on worker nodes — it bootstraps everything else
> - **kube-scheduler does not run pods** — it only decides *where* they go; kubelet does the actual work
> - Container runtime must be present on **all nodes**, including the master
