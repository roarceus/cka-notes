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

> [!tip] CKA Exam
> - Multi-container pods are common in exam tasks — watch for instructions like "add a sidecar container to the existing pod"
> - All containers in the pod share `localhost` — no need for Service objects between them
> - `kubectl logs <pod> -c <container-name>` to view logs from a specific container (see Managing Application Logs note)
> - To add a container to a running pod: edit the pod's manifest and recreate it (most pod fields are immutable — see Manual Scheduling / editing notes)
