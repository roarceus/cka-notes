# Design a Kubernetes Cluster

## Key Questions Before Designing a Cluster

- What's the **purpose**? Learning/dev/testing vs. production-grade hosting
- What's the org's **cloud adoption** — managed cloud platform or self-hosted?
- What **workloads** will run — how many applications, and what kind (web apps vs. big data/analytics)? Resource needs vary by workload type
- What **network traffic pattern** is expected — continuous heavy load vs. bursty?

---

## Cluster Types by Purpose

| Purpose | Recommended Setup | Tools |
|---|---|---|
| Learning | Minikube, or single-node cluster (local VM or cloud) | kubeadm |
| Dev/Test | Multi-node: single master + multiple workers | kubeadm, or managed (GKE/EKS/AKS) |
| Production | Highly available multi-node, **multiple master nodes** | kubeadm, kops (AWS), GKE, managed cloud |

- HA multi-master setups are covered in more detail in later lectures on high availability

---

## Production Scale Limits

- Up to **5000 nodes** per cluster
- Up to **150,000 pods** total
- Up to **300,000 containers** total
- Up to **100 pods per node**
- Node sizing depends on cluster size — cloud providers (GCP, AWS) auto-select appropriately sized nodes based on node count; on-prem deployments can use published sizing tables as a baseline

---

## Storage Considerations

- **SSD-backed storage** for high-performance workloads needing fast concurrent access
- **Network-based storage** for shared access to volumes across multiple pods
- Use **persistent volumes** and define different **storage classes**, mapping the right class to the right application

---

## Node & Master/Worker Considerations

- Nodes can be physical or virtual — cloud, on-prem, or local VMs (e.g. VirtualBox) all work
- Master nodes host control plane components (kube-apiserver, etcd, etc.); worker nodes host workloads
- Master nodes *can* technically host workloads too, but best practice (especially in production) is to **dedicate masters to control plane only**
- kubeadm enforces this by adding a **taint** to master nodes, preventing regular workloads from scheduling there
- Nodes require a **64-bit Linux OS**
- In large clusters, etcd may be **separated onto its own dedicated nodes** rather than co-located with the control plane — covered further under HA topologies

---

> [!tip] CKA Exam
> - No specific numbers (node/pod/container limits) need to be memorized for the exam — they're documented and not exam-tested
> - Know **why** masters are tainted by default (to keep control plane isolated from workloads) and that this is a best practice, not a hard requirement
> - Understand the general shape of dev/test (single master) vs. production (multi-master, HA) cluster designs
