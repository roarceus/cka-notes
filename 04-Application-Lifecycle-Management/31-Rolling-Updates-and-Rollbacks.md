# Rolling Updates and Rollbacks

## Deployment Strategies

| Strategy | Behaviour | Downtime? |
|---|---|---|
| **RollingUpdate** | Replaces pods gradually, one at a time | ❌ None — default strategy |
| **Recreate** | Kills all old pods first, then creates new ones | ✅ Yes — brief outage |

Default if not specified: **RollingUpdate**

During a rolling update, Kubernetes spins up a **new ReplicaSet** for the new version while scaling down the old one simultaneously.

---

## Rollout Commands

```bash
# Check rollout status
kubectl rollout status deployment/myapp-deployment

# View rollout history (all revisions)
kubectl rollout history deployment/myapp-deployment

# View details of a specific revision
kubectl rollout history deployment/myapp-deployment --revision=2
```

---

## Updating a Deployment

### Method 1 — Edit the YAML and apply (recommended)
```bash
# Edit image version in deployment-definition.yaml, then:
kubectl apply -f deployment-definition.yaml
```

### Method 2 — Imperative set image
```bash
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.9.1
```
> ⚠️ `kubectl set image` updates the live deployment but **does not update your YAML file** — keep them in sync manually.

---

## Rolling Back

```bash
# Undo the last rollout
kubectl rollout undo deployment/myapp-deployment

# Roll back to a specific revision
kubectl rollout undo deployment/myapp-deployment --to-revision=1
```

After a rollback, the old ReplicaSet scales back up and the new one scales down to 0.

```bash
# Verify ReplicaSet state before/after rollback
kubectl get replicasets
```

---

## How Rolling Updates Work Under the Hood

```
Old RS: 5 → 4 → 3 → 2 → 1 → 0
New RS: 0 → 1 → 2 → 3 → 4 → 5
```

Both ReplicaSets exist simultaneously during the update. This is why `kubectl get rs` shows two ReplicaSets after an update — one at 0 desired (old), one at full count (new).

---

> [!tip] CKA Exam
> - **RollingUpdate is the default** — never specify `Recreate` unless explicitly asked
> - `kubectl rollout undo` is the fastest way to revert a bad deployment
> - `kubectl set image` is quick but doesn't update the file — use `kubectl apply` after editing YAML in real scenarios
> - After an update, check `kubectl get rs` — you'll see two ReplicaSets (old scaled to 0, new at full count)
> - `kubectl rollout history` shows revision numbers — useful before rolling back to a specific version
> - Annotate changes for history: `kubectl annotate deployment myapp-deployment kubernetes.io/change-cause="updated image to 1.9.1"`
