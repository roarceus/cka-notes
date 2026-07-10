# Security Contexts

## What They Do

Security contexts configure security settings for pods or containers — controlling things like which user a process runs as, Linux capabilities, and privilege escalation.

---

## Pod-level vs Container-level

- **Pod-level** (`spec.securityContext`) — applies to **all containers** in the pod
- **Container-level** (`spec.containers[].securityContext`) — applies to **one container only**
- Container-level settings **override** pod-level settings when both are set

---

## YAML Examples

### Pod-level (applies to all containers)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000        # all containers run as UID 1000
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
```

### Container-level (overrides pod-level)

```yaml
spec:
  securityContext:
    runAsUser: 1000          # pod-level default
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 2000        # overrides pod-level for this container
      capabilities:
        add: ["MAC_ADMIN"]   # Linux capability — container-level ONLY
        drop: ["ALL"]
```

> ⚠️ **`capabilities`** can only be set at the **container level** — not pod level.

---

## Common Security Context Fields

| Field | Level | Description |
|---|---|---|
| `runAsUser` | Pod / Container | Run as specific UID |
| `runAsGroup` | Pod / Container | Run as specific GID |
| `runAsNonRoot` | Pod / Container | Enforce non-root user |
| `readOnlyRootFilesystem` | Container | Make root filesystem read-only |
| `allowPrivilegeEscalation` | Container | Allow/deny privilege escalation |
| `capabilities.add` | Container only | Add Linux capabilities |
| `capabilities.drop` | Container only | Drop Linux capabilities |
| `privileged` | Container | Run as privileged container |

---

> [!tip] CKA Exam
> - `capabilities` is **container-level only** — putting it under pod `securityContext` will error
> - Container-level `securityContext` overrides pod-level for that container
> - `runAsNonRoot: true` causes the pod to fail if the image defaults to root — combine with `runAsUser`
> - To check which user a container is running as: `kubectl exec <pod> -- whoami`
