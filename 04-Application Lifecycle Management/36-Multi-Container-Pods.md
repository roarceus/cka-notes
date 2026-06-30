# Multi-Container Pods

## Why Use Them

When two services are **tightly coupled** and need to scale together (created/destroyed as a pair), put them in the same pod rather than wiring up separate pods with shared volumes and networking manually.

Classic example: a web server + a dedicated logging agent (sidecar pattern) — each web server instance gets its own log shipper, and they live and die together.

---

## What They Share

| Resource | Shared? |
|---|---|
| Lifecycle | ✅ Created and terminated together |
| Network namespace | ✅ Communicate via `localhost` |
| Storage volumes | ✅ Can mount the same volume |

This eliminates the need to configure cross-pod networking or shared volumes between separate pods.

---

## YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent       # second container — same pod
    image: log-agent
```

`containers` is an **array** — just add another item to include more containers in the same pod.

---

---

## Container Restart Behaviour (Important Nuance)

`restartPolicy` applies at the **container level**, not the pod level. If one container in a multi-container pod exits, only that container is restarted per the policy — other containers keep running undisturbed.

| Policy | Behaviour |
|---|---|
| `Always` (default) | Restarts container after any exit, regardless of exit code |
| `OnFailure` | Restarts only on non-zero exit code |
| `Never` | Never restarts |

> Kubernetes does **not** restart the whole pod when one container fails — the pod is only recreated by its controller (Deployment, etc.) if the node dies or the pod is deleted.

---

## Multi-Container Design Patterns

There are three distinct patterns — easy to confuse the first and third, so pay attention to the difference.

### 1. Co-located containers (basic form)
Both containers run for the entire pod lifecycle, started together with **no guaranteed order**.

```yaml
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
  - name: log-agent
    image: log-agent
```
Use when there's no strict startup-order requirement.

### 2. Init Containers
Run **before** the main app, in the `initContainers` section. Each must succeed (**exit 0**) before the next one starts; once all complete, the regular containers start simultaneously.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:
  - name: init-myservice
    image: busybox:1.31
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.31
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

- Multiple init containers run **sequentially**, in the order listed
- If **any** init container fails, the **entire pod restarts** and all init containers rerun from the beginning

### 3. Sidecar Containers — Native Support (Kubernetes v1.33+)

Since **v1.33**, sidecars are natively supported — no more entrypoint hacks needed. Declared in `initContainers` but with `restartPolicy: Always` set **on the container itself**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
  - name: sidecar-logger
    image: busybox:1.31
    restartPolicy: Always       # ← marks this as a native sidecar
    command: ['sh', '-c', 'while true; do echo Sidecar running; sleep 10; done']
  containers:
  - name: main-app
    image: busybox:1.31
    command: ['sh', '-c', 'echo Main app starting; sleep 60']
```

Kubernetes treats this container as a sidecar: it starts first, runs alongside the main app for the full pod lifecycle, and shuts down after the main app completes.

> Sidecar containers (with their own `restartPolicy: Always`) restart **regardless of the pod's restartPolicy** — this overrides the normal init container "fail once, rerun everything" behaviour.

**Real-world example:** Elasticsearch + Kibana logging stack — a **Filebeat** sidecar starts before the main app (to capture startup logs) and keeps running until the main app terminates (to capture shutdown/error logs).

---

## Co-located vs Sidecar — The Key Difference

| | Co-located | Native Sidecar |
|---|---|---|
| Startup order | ❌ No guarantee — both start together | ✅ Guaranteed — sidecar starts first |
| Defined under | `containers` (as a 2nd item) | `initContainers` with `restartPolicy: Always` |
| Use case | No ordering needed | Must capture full app lifecycle (startup → shutdown) |

---

> [!tip] CKA Exam
> - Multi-container pods are common in exam tasks — watch for instructions like "add a sidecar container to the existing pod"
> - All containers in the pod share `localhost` — no need for Service objects between them
> - `kubectl logs <pod> -c <container-name>` to view logs from a specific container (see Managing Application Logs note)
> - To add a container to a running pod: edit the pod's manifest and recreate it (most pod fields are immutable — see Manual Scheduling / editing notes)
> - **Init containers run sequentially** — each must exit 0 before the next starts; one failure restarts the *whole pod* and reruns *all* init containers
> - **Native sidecars (v1.33+)** are defined in `initContainers` but with `restartPolicy: Always` on the container itself — check the cluster version before assuming this feature is available
> - `restartPolicy` is a **container-level** concept inside multi-container pods — one container failing doesn't restart its siblings
