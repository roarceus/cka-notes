# Manual Scheduling

## How the Scheduler Normally Works

Every pod has a `nodeName` field — empty by default. The scheduler finds pods with no `nodeName`, picks a node, and sets this field. If there's **no scheduler running**, pods stay stuck in `Pending`.

---

## Method 1 — Set `nodeName` at Creation

Add `nodeName` directly to the pod spec before creating it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: node02        # manually assign to this node
  containers:
  - name: nginx
    image: nginx
```

> ⚠️ `nodeName` can only be set **at creation time** — you cannot edit it on a running pod.

---

## Method 2 — Binding Object (for already-running pods)

If a pod is already created (and stuck in `Pending`), use a Binding object to assign it:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

Send it via the API directly:
```bash
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"apiVersion":"v1","kind":"Binding","metadata":{"name":"nginx"},"target":{"apiVersion":"v1","kind":"Node","name":"node02"}}' \
  http://$SERVER/api/v1/namespaces/default/pods/nginx/binding
```

---

> [!tip] CKA Exam
> - If a pod is stuck in `Pending` and there's no scheduler, check for missing `nodeName`
> - The simplest fix: **delete and recreate** the pod with `nodeName` set
> - You cannot `kubectl edit` a running pod's `nodeName` — it's an immutable field
> - In practice, the Binding API approach is rarely used in the exam — recreating the pod is faster
