# Commands and Arguments in Docker

## Why This Matters

Understanding Docker `CMD` vs `ENTRYPOINT` is essential because these map directly to `command` and `args` in Kubernetes pod specs.

---

## CMD vs ENTRYPOINT

| | `CMD` | `ENTRYPOINT` |
|---|---|---|
| Purpose | Default command/args to run | The fixed executable to run |
| Runtime override | Completely **replaced** by args passed to `docker run` | Args are **appended** to it |
| Override flag | *(just pass new command)* | `--entrypoint` flag |

---

## How They Work Together

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]   # fixed executable
CMD ["5"]              # default argument — overridable
```

| Command | Result |
|---|---|
| `docker run ubuntu-sleeper` | `sleep 5` |
| `docker run ubuntu-sleeper 10` | `sleep 10` (overrides CMD) |
| `docker run --entrypoint sleep2.0 ubuntu-sleeper 10` | `sleep2.0 10` (overrides ENTRYPOINT) |

---

## CMD Formats

```dockerfile
# Shell form (runs in /bin/sh -c)
CMD sleep 5

# JSON array / exec form (preferred — no shell wrapper)
CMD ["sleep", "5"]
```

> Always use the **JSON array format** in production — it's more explicit and doesn't invoke a shell.

---

> [!tip] CKA Exam
> - `ENTRYPOINT` → maps to `command` in Kubernetes pod spec
> - `CMD` → maps to `args` in Kubernetes pod spec
> - This is covered in the next note — knowing the Docker mapping is the key prerequisite

---

## In Kubernetes Pod Spec

### The mapping

| Docker | Kubernetes pod spec | Effect |
|---|---|---|
| `ENTRYPOINT` | `command` | Overrides the container's entrypoint entirely |
| `CMD` | `args` | Overrides the default arguments |

### Overriding CMD only (most common)

```yaml
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper    # ENTRYPOINT ["sleep"], CMD ["5"]
    args: ["10"]             # overrides CMD → runs: sleep 10
```

### Overriding both ENTRYPOINT and CMD

```yaml
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]    # overrides ENTRYPOINT
    args: ["10"]             # overrides CMD → runs: sleep2.0 10
```

> ⚠️ `command` in a pod spec **completely replaces** the Dockerfile `ENTRYPOINT` — it doesn't append to it.
