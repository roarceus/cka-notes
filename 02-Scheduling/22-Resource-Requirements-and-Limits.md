# Resource Requirements and Limits

## Concepts

- **Request** — the minimum CPU/memory a container needs. The scheduler uses this to find a suitable node.
- **Limit** — the maximum CPU/memory a container is allowed to consume.
- Set per **container**, not per pod. Each container in a pod can have different values.

---

## CPU Units

| Value | Meaning |
|---|---|
| `1` | 1 vCPU (AWS), 1 Core (GCP/Azure), 1 Hyperthread |
| `0.1` | 100m (milli-CPU) |
| `1m` | Minimum possible (cannot go lower) |

## Memory Units

| Suffix | Type | Equivalent |
|---|---|---|
| `G` | Gigabyte | 1,000,000,000 bytes |
| `Gi` | Gibibyte | 1,073,741,824 bytes |
| `M` | Megabyte | 1,000,000 bytes |
| `Mi` | Mebibyte | 1,048,576 bytes |
| `K` | Kilobyte | 1,000 bytes |
| `Ki` | Kibibyte | 1,024 bytes |

> Use `Mi` and `Gi` in practice — they're the standard in Kubernetes YAML.

---

## YAML — Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    resources:
      requests:
        memory: "4Gi"
        cpu: 2
      limits:
        memory: "2Gi"
        cpu: 2
```

---

## CPU vs Memory Behaviour at the Limit

| Resource | At limit behaviour |
|---|---|
| **CPU** | **Throttled** — cannot exceed limit, but pod keeps running |
| **Memory** | **OOMKilled** — pod is terminated if it consistently exceeds limit |

> `OOMKilled` = Out Of Memory Kill. Check with `kubectl describe pod <name>` — you'll see the termination reason.

---

## The 4 Scenarios (CPU & Memory behave the same way)

| Requests | Limits | Result |
|---|---|---|
| ❌ None | ❌ None | Any pod can consume all node resources — dangerous |
| ❌ None | ✅ Set | Kubernetes sets requests = limits automatically. Each pod gets a guaranteed fixed amount |
| ✅ Set | ✅ Set | Pod gets guaranteed minimum, can burst up to limit |
| ✅ Set | ❌ None | ✅ **Ideal for CPU** — guaranteed minimum, can use available cycles freely |

**Key difference for memory:** with no limits, a pod can consume all memory. Unlike CPU (which throttles), memory can't be reclaimed without killing the pod. So for memory, setting limits is more important.

> **Recommended defaults:**
> - CPU: set requests, skip limits (allows bursting)
> - Memory: set both requests and limits (prevents OOMKill spirals)

---

## LimitRange — Namespace-level Defaults

Automatically applies default requests/limits to any pod created **without** explicit values. Enforced at creation time — does not affect existing pods.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - type: Container
    default:            # default limit
      cpu: 500m
    defaultRequest:     # default request
      cpu: 500m
    max:                # max limit allowed
      cpu: "1"
    min:                # min request allowed
      cpu: 100m
```

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - type: Container
    default:
      memory: 1Gi
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
```

---

## ResourceQuota — Namespace-level Hard Caps

Limits the **total** resources all pods in a namespace can consume combined.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

---

> [!tip] CKA Exam
> - `OOMKilled` in pod status = container exceeded memory limit
> - CPU exceeding limit = **throttled** (pod stays running); memory exceeding limit = **killed**
> - `LimitRange` sets defaults per container; `ResourceQuota` caps the whole namespace
> - LimitRange changes **do not** affect already-running pods
> - If a pod is `Pending` with event `Insufficient CPU/memory`, the node doesn't have enough resources to satisfy the request
> - Check resource usage: `kubectl top pod` / `kubectl top node` (requires metrics-server)
