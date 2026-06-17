# Node Affinity

## What It Does

Node Affinity is the advanced alternative to `nodeSelector`. It supports complex expressions for pod-to-node scheduling rules.

| Feature | nodeSelector | Node Affinity |
|---|---|---|
| Simple key=value match | ✅ | ✅ |
| OR logic (Large or Medium) | ❌ | ✅ |
| NOT logic (not Small) | ❌ | ✅ |
| Label existence check | ❌ | ✅ |

---

## Affinity Types

| Type | During Scheduling | During Execution |
|---|---|---|
| `requiredDuringSchedulingIgnoredDuringExecution` | Must match — pod won't schedule otherwise | Label changes ignored |
| `preferredDuringSchedulingIgnoredDuringExecution` | Tries to match — falls back to any node | Label changes ignored |

> A future type `requiredDuringSchedulingRequiredDuringExecution` will **evict** pods if node labels change and no longer match — not yet available.

---

## Operators

| Operator | Meaning |
|---|---|
| `In` | Label value is in the list |
| `NotIn` | Label value is NOT in the list |
| `Exists` | Label key exists (no value needed) |
| `DoesNotExist` | Label key does not exist |
| `Gt` / `Lt` | Greater/less than (numeric) |

---

## YAML Examples

### Required — schedule only on Large nodes
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values:
          - Large
```

### Required — Large or Medium
```yaml
        - key: size
          operator: In
          values:
          - Large
          - Medium
```

### Required — anything except Small
```yaml
        - key: size
          operator: NotIn
          values:
          - Small
```

### Required — node just needs the label (any value)
```yaml
        - key: size
          operator: Exists
```

### Preferred — try Large, but schedule anywhere if needed
```yaml
affinity:
  nodeAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
        matchExpressions:
        - key: size
          operator: In
          values:
          - Large
```

---

## Taints/Tolerations vs Node Affinity vs Combined

| Mechanism | What it does | What it doesn't do |
|---|---|---|
| Taints & Tolerations | Repels pods from a node unless they tolerate the taint | Doesn't guarantee the pod lands on *that specific* node |
| Node Affinity | Attracts pods to nodes with matching labels | Doesn't stop *other* pods landing on the same node |
| **Both combined** | ✅ Exclusive node dedication — pods go to the right node AND other pods are kept off | — |

**Use both together** when you need a node dedicated exclusively to specific workloads (e.g. multi-tenant clusters, GPU nodes).

---

> [!tip] CKA Exam
> - `required` = hard rule — pod stays `Pending` if no node matches
> - `preferred` = soft rule — pod will schedule somewhere even if no match
> - The long field names are easy to mistype — use `--dry-run=client -o yaml` and edit rather than writing from scratch
> - For **exclusive node assignment**: apply a taint to the node + add toleration to the pod + add node affinity to the pod
> - Label nodes first: `kubectl label nodes <node> size=Large`
