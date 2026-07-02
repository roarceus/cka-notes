# Vertical Pod Autoscaler (VPA)

## What It Does

VPA automatically **adjusts CPU and memory requests/limits** for running pods based on actual usage — vertical scaling (bigger pods, not more pods). Not built into Kubernetes by default; must be installed separately.

---

## VPA Components

| Component | Role |
|---|---|
| **VPA Recommender** | Monitors metrics and generates recommended resource values |
| **VPA Updater** | Evicts pods that are running with suboptimal resources so they can be recreated with updated values |
| **VPA Admission Controller** | Intercepts new pod creation and mutates the spec with recommended values |

---

## Installing VPA

```bash
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml

# Verify components are running
kubectl get pods -n kube-system | grep vpa
# vpa-admission-controller-xxxx   Running
# vpa-recommender-xxxx            Running
# vpa-updater-xxxx                Running
```

---

## VPA YAML (declarative only — no imperative command)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"       # Auto | Recreate | Initial | Off
  resourcePolicy:
    containerPolicies:
    - containerName: my-app
      minAllowed:
        cpu: "250m"
      maxAllowed:
        cpu: "2"
      controlledResources: ["cpu"]
```

### Update Modes

| Mode | Behaviour |
|---|---|
| `Auto` | Evicts and recreates pods with updated resources |
| `Recreate` | Same as Auto currently |
| `Initial` | Only sets resources at pod creation — no updates to running pods |
| `Off` | Recommendations only — no automatic changes |

---

## Checking VPA Recommendations

```bash
kubectl describe vpa my-app-vpa
# Shows: Target cpu: 1.5 (recommended value)
```

---

## VPA vs HPA

| | VPA | HPA |
|---|---|---|
| Scaling method | Bigger pods (more CPU/memory) | More pods |
| Pod restart needed | ✅ Yes (currently — until in-place resize is stable) | ❌ No |
| Traffic spikes | ⚠️ Slow — restart delay | ✅ Fast — adds pods instantly |
| Best for | Stateful apps, databases, JVM/AI workloads | Stateless services, web apps, microservices |
| Can run together? | ⚠️ Careful — don't use both on the same resource (e.g. both managing CPU) |  |

---

> [!tip] CKA Exam
> - VPA is **not built in** — must be installed; `apiVersion: autoscaling.k8s.io/v1`
> - No imperative command — VPA is **declarative only** (unlike HPA which has `kubectl autoscale`)
> - `updateMode: "Off"` is useful for just getting recommendations without any automation
> - In `Auto` mode, VPA **evicts and recreates pods** — this causes brief downtime (unlike HPA)
> - Don't use VPA and HPA on the same resource (e.g. both managing CPU) — they'll conflict
