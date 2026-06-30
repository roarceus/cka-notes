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
Run **before** the main app, then **exit**. Used for setup tasks (e.g. wait for a DB to be ready).

```yaml
spec:
  initContainers:
  - name: init-wait-for-db
    image: busybox
    command: ['sh', '-c', 'wait-for-db.sh']
  containers:
  - name: main-app
    image: main-app
```

- Multiple init containers run **sequentially**, in the order listed
- Each must complete successfully before the next (or the main container) starts
- If an init container fails, the pod restarts it (per pod's `restartPolicy`)

### 3. Sidecar Containers (proper definition)
Like an init container, but **keeps running** alongside the main app for the whole pod lifecycle. Starts first, runs throughout, and is terminated after the main app stops — so it can capture both startup and shutdown logs.

> Mechanically this still uses the `initContainers`-style ordering, but the sidecar is configured with `restartPolicy: Always` so it doesn't exit after its initial run.

**Real-world example:** Elasticsearch + Kibana logging stack — a **Filebeat** sidecar starts before the main app (to capture startup logs) and keeps running until the main app terminates (to capture shutdown/error logs).

---

## Co-located vs Sidecar — The Key Difference

| | Co-located | Sidecar |
|---|---|---|
| Startup order | ❌ No guarantee — both start together | ✅ Guaranteed — sidecar starts first |
| Defined under | `containers` (as a 2nd item) | `initContainers` pattern with `restartPolicy: Always` |
| Use case | No ordering needed | Must capture full app lifecycle (startup → shutdown) |

---

> [!tip] CKA Exam
> - Multi-container pods are common in exam tasks — watch for instructions like "add a sidecar container to the existing pod"
> - All containers in the pod share `localhost` — no need for Service objects between them
> - `kubectl logs <pod> -c <container-name>` to view logs from a specific container (see Managing Application Logs note)
> - To add a container to a running pod: edit the pod's manifest and recreate it (most pod fields are immutable — see Manual Scheduling / editing notes)
