# Priority Classes

## What They Do

Priority classes assign a **numerical priority** to pods. Higher value = scheduled first. If resources are scarce, higher-priority pods can **preempt** (evict) lower-priority ones.

---

## Priority Value Ranges

| Scope | Range |
|---|---|
| User workloads | ~ -2 billion to +1 billion |
| System-critical (reserved) | Up to ~2 billion |

### Built-in system priority classes

```bash
kubectl get priorityclass
# NAME                      VALUE          PREEMPTIONPOLICY
# system-cluster-critical   2000000000     PreemptLowerPriority
# system-node-critical      2000010000     PreemptLowerPriority
```

---

## Creating a Priority Class

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
globalDefault: false          # set true to make this the default for all pods
preemptionPolicy: PreemptLowerPriority   # default behaviour
```

- Only **one** priority class can have `globalDefault: true`
- Pods without a `priorityClassName` get priority **0** by default

---

## Using a Priority Class in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: high-priority    # reference the PriorityClass name
```

---

## Preemption Policy

| Policy | Behaviour |
|---|---|
| `PreemptLowerPriority` | Default — evicts lower-priority pods to free resources |
| `Never` | Pod waits in queue — does not evict anyone |

```yaml
preemptionPolicy: Never    # higher priority pod waits rather than evicting
```

---

> [!tip] CKA Exam
> - `apiVersion: scheduling.k8s.io/v1` — unique to PriorityClass
> - Pods with no `priorityClassName` get priority **0**
> - Only **one** `globalDefault: true` priority class allowed per cluster
> - `PreemptLowerPriority` is the default — pods can be evicted to make room for higher-priority ones
> - Check existing priority classes: `kubectl get priorityclass` (or `kubectl get pc`)
