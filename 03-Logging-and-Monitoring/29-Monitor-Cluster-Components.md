# Monitor Cluster Components

## What to Monitor

- **Node level** — number of nodes, health, CPU, memory, network, disk
- **Pod level** — number of pods, CPU and memory per pod

---

## Monitoring Solutions

Kubernetes has no full built-in monitoring solution. Common options:

| Solution | Type |
|---|---|
| **Metrics Server** | Lightweight, in-memory — built for Kubernetes |
| Prometheus | Open source, full-featured |
| Elastic Stack | Open source |
| Datadog / Dynatrace | Proprietary |

> **Heapster** was the original monitoring project — now deprecated. Metrics Server is its slimmed-down replacement.

---

## Metrics Server

- **One per cluster**
- Retrieves metrics from each node and pod via the kubelet's **cAdvisor** (Container Advisor) subcomponent
- Stores data **in memory only** — no historical data
- For historical metrics, use Prometheus or another full solution

### How metrics flow
```
Pod → cAdvisor (inside kubelet) → kubelet API → Metrics Server
```

### Installing Metrics Server

```bash
# Minikube
minikube addons enable metrics-server

# All other clusters
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## Viewing Metrics

```bash
# Node resource usage
kubectl top node

# Pod resource usage
kubectl top pod

# Pod metrics in a specific namespace
kubectl top pod -n kube-system

# Sort by CPU or memory
kubectl top pod --sort-by=cpu
kubectl top pod --sort-by=memory
```

---

> [!tip] CKA Exam
> - `kubectl top node` / `kubectl top pod` require **metrics-server** to be installed — if it returns an error, the server isn't running
> - Metrics Server stores data **in memory only** — no historical data available from it
> - cAdvisor runs inside the kubelet on every node — it's the source of all pod/container metrics
> - If asked to find which pod/node consumes the most CPU or memory, use `kubectl top` with `--sort-by`
