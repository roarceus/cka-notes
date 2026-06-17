# Imperative vs Declarative

## The Core Difference

| | Imperative | Declarative |
|---|---|---|
| **How** | You specify *what to do* step by step | You specify *desired end state*; Kubernetes figures out the steps |
| **Tool** | `kubectl run`, `create`, `edit`, `scale`, `delete` | `kubectl apply -f file.yaml` |
| **Best for** | Quick one-off tasks, exam speed | Production, team environments, version control |
| **Idempotent?** | ❌ Running twice may error | ✅ Safe to run repeatedly |

---

## Imperative Commands (quick reference)

```bash
# Create objects
kubectl run nginx --image=nginx
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80

# Modify live objects
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18

# From file (imperative file-based)
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml
```

⚠️ `kubectl create` errors if the object already exists. `kubectl replace` errors if it doesn't exist yet.

---

## Declarative (apply)

```bash
# Create or update — always safe to re-run
kubectl apply -f nginx.yaml

# Apply an entire directory of manifests
kubectl apply -f /path/to/configs/
```

`kubectl apply` compares the file against the **last applied config** stored as an annotation on the live object, then makes only the necessary changes.

---

## The `kubectl edit` Trap

`kubectl edit` modifies the **live object** in the cluster — changes are not saved back to your local YAML file. If you later run `kubectl apply -f nginx.yaml`, your live edits may be overwritten.

**Safe workflow:**
1. Edit your local YAML file
2. `kubectl apply -f nginx.yaml` (declarative) or `kubectl replace -f nginx.yaml` (imperative)

---

## How `kubectl apply` Works Internally

`kubectl apply` does a **3-way merge** between:
1. Your **local YAML file**
2. The **live object** in the cluster
3. The **last applied configuration** (stored as a JSON annotation on the live object)

```yaml
# annotation stored on every object created with kubectl apply
kubectl.kubernetes.io/last-applied-configuration: '{"apiVersion":"v1",...}'
```

This is how it handles each case:

| Scenario | Result |
|---|---|
| Field changed in local file | Updates the live object |
| Field added in local file | Adds it to the live object |
| Field removed from local file | Removes it from the live object (detected via last-applied annotation) |
| Object doesn't exist yet | Creates it |

> ⚠️ **Don't mix imperative and declarative.** `kubectl create` / `kubectl replace` do **not** write the last-applied annotation. If you then run `kubectl apply`, the 3-way merge breaks because the annotation is missing or stale.

---

> [!tip] CKA Exam
> - **Imperative for speed** — use `kubectl run`, `create`, `expose` to create objects fast
> - **Declarative for edits** — modify the YAML and `kubectl apply` rather than `kubectl edit`
> - `kubectl apply` is the safest all-rounder: creates if absent, updates if present
> - `--dry-run=client -o yaml` bridges both worlds: generate YAML imperatively, apply declaratively
> - In the exam, `kubectl apply` is generally preferred over `kubectl create`/`kubectl replace` to avoid "already exists" errors
