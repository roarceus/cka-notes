# Node Selectors

## What They Do

`nodeSelector` restricts a pod to only run on nodes that have **matching labels**. The simplest form of node-targeted scheduling.

---

## How to Use It

### Step 1 — Label the node
```bash
kubectl label nodes node-1 size=Large
```

### Step 2 — Add `nodeSelector` to the pod spec
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large       # must match a label on the target node
```

---

## Limitations

`nodeSelector` only supports **exact single-label matching**. It cannot express:
- "Large **or** Medium"
- "Anything **except** Small"
- Complex multi-condition logic

→ Use **Node Affinity** for these cases (covered in the next note).

---

> [!tip] CKA Exam
> - Always label the node **before** creating the pod, or it will stay `Pending`
> - Check node labels: `kubectl get nodes --show-labels`
> - Add a label to a node: `kubectl label nodes <node-name> key=value`
> - Remove a node label: `kubectl label nodes <node-name> key-`
