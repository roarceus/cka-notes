# Autoscaling in Kubernetes

## Scaling Dimensions

|  | Horizontal | Vertical |
|---|---|---|
| **Cluster nodes** | Add more nodes (`kubeadm join`) | Increase CPU/memory on existing nodes |
| **Workloads (pods)** | Add more pods | Increase resource requests/limits on existing pods |

---

## Manual Scaling

```bash
# Add a node to the cluster
kubeadm join ...

# Scale pods horizontally
kubectl scale --replicas=5 deployment/myapp

# Scale pods vertically (edit resource requests/limits)
kubectl edit deployment/myapp
```

---

## Automated Scaling

| Autoscaler | What it manages |
|---|---|
| **Horizontal Pod Autoscaler (HPA)** | Automatically scales pod count based on metrics (CPU, memory, custom) |
| **Vertical Pod Autoscaler (VPA)** | Automatically adjusts resource requests/limits on existing pods |
| **Cluster Autoscaler** | Automatically adds/removes nodes from the cluster |

> HPA and VPA are covered in the next notes. Both require **metrics-server** (or a custom metrics provider) to function.

---

> [!tip] CKA Exam
> - Know the difference: **HPA = more pods**, **VPA = bigger pods**
> - Vertical node scaling is uncommon in Kubernetes — usually easier to add a new larger node and drain the old one
