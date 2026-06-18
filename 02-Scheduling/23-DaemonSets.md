# DaemonSets

## What They Do

A DaemonSet ensures **exactly one pod runs on every node** in the cluster. When a node is added, the pod is automatically deployed to it. When a node is removed, the pod is cleaned up.

---

## Common Use Cases

| Use Case | Example |
|---|---|
| Monitoring / logging | Prometheus node-exporter, Fluentd |
| Networking agents | weave-net, Calico, Flannel |
| Essential cluster components | kube-proxy (deployed as a DaemonSet by kubeadm) |

---

## DaemonSet YAML

Nearly identical to a ReplicaSet — just change `kind` and drop `replicas`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent    # must match selector
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

```bash
kubectl create -f daemon-set-definition.yaml

# List DaemonSets
kubectl get daemonsets
kubectl get ds             # shorthand

# Describe
kubectl describe daemonset monitoring-daemon
```

---

## How DaemonSets Schedule Pods

- **Before v1.12:** set `nodeName` directly on each pod
- **v1.12+:** uses the default scheduler with **node affinity rules** — fully automatic

---

> [!tip] CKA Exam
> - DaemonSet `apiVersion` is **`apps/v1`** — same as Deployment and ReplicaSet
> - No `replicas` field — one pod per node is implicit
> - `selector.matchLabels` must match `template.metadata.labels` — same rule as ReplicaSet
> - kube-proxy is itself a DaemonSet — `kubectl get ds -n kube-system` to see it
> - DaemonSets **ignore** `nodeName` / `nodeSelector` on the pod template for scheduling — they manage placement themselves via node affinity
> - To create quickly: there's no `kubectl create daemonset` imperative command — use `--dry-run=client -o yaml` from a Deployment and change `kind: DaemonSet`, remove `replicas` and `strategy`
