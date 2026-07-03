# OS Upgrades — Node Maintenance

## What Happens When a Node Goes Down

- Node down **< 5 minutes** → pods restart when the node comes back; cluster waits
- Node down **> 5 minutes** → Kubernetes marks pods as dead:
  - Pods managed by a ReplicaSet/Deployment → recreated on other nodes
  - Standalone pods (no controller) → **lost permanently**, not rescheduled

---

## The Three Commands

| Command | What it does |
|---|---|
| `kubectl drain <node>` | Evicts all pods gracefully, then marks node unschedulable (cordon) |
| `kubectl cordon <node>` | Marks node unschedulable — existing pods stay, no new pods scheduled |
| `kubectl uncordon <node>` | Marks node schedulable again — pods don't auto-return, but new ones can land here |

---

## Drain vs Cordon

- **Drain** = evict pods + cordon. Use before taking a node offline for maintenance.
- **Cordon** = unschedulable only, existing pods untouched. Use when you want to stop new scheduling without disrupting current workloads.

---

## Safe Node Maintenance Workflow

```bash
# 1. Drain the node (evicts pods + cordons node)
kubectl drain node-1 --ignore-daemonsets

# 2. Perform maintenance (OS patch, reboot, etc.)

# 3. Bring node back online, then uncordon
kubectl uncordon node-1
```

> Pods that were evicted to other nodes **do not automatically return** after uncordoning — they stay where they were rescheduled.

---

## Common Drain Flags

```bash
# Ignore DaemonSet pods (required if node has daemonset pods — can't evict them)
kubectl drain node-1 --ignore-daemonsets

# Force eviction of standalone pods (not managed by a controller) — they will be lost
kubectl drain node-1 --ignore-daemonsets --force

# Grace period override
kubectl drain node-1 --ignore-daemonsets --grace-period=0
```

---

> [!tip] CKA Exam
> - Always use `--ignore-daemonsets` with `kubectl drain` — otherwise it errors on DaemonSet pods
> - If there are standalone pods not owned by a controller, you also need `--force` — and those pods will be gone
> - After `kubectl uncordon`, check the node is `Ready` and `SchedulingEnabled`: `kubectl get nodes`
> - `kubectl cordon` alone doesn't touch running pods — use it when you just want to stop new scheduling
> - Node status after cordon/drain shows `SchedulingDisabled` in `kubectl get nodes`
