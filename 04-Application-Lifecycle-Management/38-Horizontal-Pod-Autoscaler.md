# Horizontal Pod Autoscaler (HPA)

## What It Does

HPA automatically **scales pod count** up or down based on real-time metrics (CPU, memory, or custom). It monitors a Deployment, StatefulSet, or ReplicaSet and adjusts replicas to keep utilisation at a target threshold.

Requires **metrics-server** to be installed.

---

## Imperative — Fastest for Exam

```bash
# Create HPA targeting 50% CPU, min 1 pod, max 10 pods
kubectl autoscale deployment my-app --cpu-percent=50 --min=1 --max=10

# View HPA status
kubectl get hpa

# Delete HPA
kubectl delete hpa my-app
```

---

## Declarative YAML (`autoscaling/v2`)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50   # scale up when avg CPU > 50% of limit
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
```

---

## How It Works

1. metrics-server collects CPU/memory usage from each pod
2. HPA calculates: `desired replicas = current replicas × (current utilisation / target utilisation)`
3. HPA updates the deployment's replica count accordingly
4. Scales down only after a cooldown period (to avoid flapping)

---

## Metrics Sources

| Source | Examples |
|---|---|
| Resource metrics (built-in) | CPU, memory via metrics-server |
| Custom metrics | App-specific metrics via custom adapter |
| External metrics | Datadog, Dynatrace via external adapter |

---

> [!tip] CKA Exam
> - `kubectl autoscale deployment <name> --cpu-percent=<n> --min=<n> --max=<n>` — fastest imperative approach
> - HPA `apiVersion` is **`autoscaling/v2`** (v1 is deprecated)
> - HPA won't work without **metrics-server** — if `kubectl get hpa` shows `<unknown>` for targets, metrics-server is missing
> - The deployment must have **resource requests set** — HPA calculates utilisation as `current usage / request`, not `current usage / limit`
> - `kubectl get hpa` shows current vs target utilisation and current replica count in real time
