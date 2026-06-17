# Taints and Tolerations

## Concept

- **Taint** — applied to a **node** to repel pods from being scheduled there
- **Toleration** — applied to a **pod** to allow it to be scheduled on a tainted node

> Taints and tolerations control *which pods can land on a node* — they do **not** guarantee a pod goes to a specific node. For that, use **Node Affinity**.

---

## Taint Effects

| Effect | Behaviour |
|---|---|
| `NoSchedule` | New pods without matching toleration will not be scheduled on the node |
| `PreferNoSchedule` | Scheduler tries to avoid placing non-tolerating pods, but not enforced |
| `NoExecute` | Non-tolerating pods are not scheduled **and** any already running are evicted |

---

## Tainting a Node

```bash
# Format: kubectl taint nodes <node-name> key=value:effect
kubectl taint nodes node1 app=blue:NoSchedule

# Remove a taint (add - at the end)
kubectl taint nodes node1 app=blue:NoSchedule-

# Check taints on a node
kubectl describe node node1 | grep Taint
```

---

## Adding a Toleration to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

All four fields (`key`, `operator`, `value`, `effect`) must match the taint exactly.

---

## Master Node Taint

By default, the master node has a taint that prevents workload pods from being scheduled on it:

```bash
kubectl describe node kubemaster | grep Taint
# Taints: node-role.kubernetes.io/master:NoSchedule
```

This is why pods never land on the control plane node unless you explicitly tolerate this taint.

---

> [!tip] CKA Exam
> - Taints repel pods; tolerations grant exceptions — **not** a guarantee of placement
> - To remove a taint: same command with `-` appended → `kubectl taint nodes node1 app=blue:NoSchedule-`
> - `NoExecute` is the most aggressive — evicts **running** pods too
> - The master node taint is `node-role.kubernetes.io/control-plane:NoSchedule` in newer clusters (replaced `master`)
> - If a pod is stuck `Pending`, check if the target node has taints the pod doesn't tolerate: `kubectl describe node <name> | grep Taint`
