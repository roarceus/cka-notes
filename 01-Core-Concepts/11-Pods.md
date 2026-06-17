# Pods

## What is a Pod?

A pod is the **smallest deployable unit in Kubernetes**. It wraps one or more containers and represents a single instance of an application.

- You don't run containers directly — Kubernetes always wraps them in a pod
- One pod = one instance of your app (usually one container)
- To scale → **add more pods**, not more containers inside a pod

---

## Single vs Multi-Container Pods

### Single container (most common)
One pod, one container. Simple and standard.

### Multi-container pods
Multiple containers in one pod that are **complementary**, not redundant (e.g. a main app + a helper/sidecar).

Containers in the same pod share:
- **Network namespace** → communicate via `localhost`
- **Storage volumes**
- **Lifecycle** → start and stop together

![Multi-container pod](https://kodekloud.com/kk-media/image/upload/v1752869733/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Pods/frame_230.jpg)

---

## Basic Pod Commands

```bash
# Create and run a pod (imperative)
kubectl run nginx --image=nginx

# List pods
kubectl get pods

# More detail (node, IP, status)
kubectl get pods -o wide

# Describe a pod (events, image, state)
kubectl describe pod nginx

# Delete a pod
kubectl delete pod nginx
```

---

## Pod Definition (YAML)

### The 4 required top-level fields

| Field | What it is |
|---|---|
| `apiVersion` | API version for the object type |
| `kind` | Object type (Pod, Deployment, Service…) |
| `metadata` | Name, namespace, labels — a dictionary |
| `spec` | Object-specific config (containers, volumes…) |

### Common `apiVersion` values

| Kind | apiVersion |
|---|---|
| Pod | `v1` |
| Service | `v1` |
| ReplicaSet | `apps/v1` |
| Deployment | `apps/v1` |

### Example Pod manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container    # name of the container
    image: nginx             # Docker image to use
```

> `containers` is a **list** (note the `-` dash) — a pod can have multiple containers
> YAML indentation: **2 spaces per level, never tabs**

```bash
# Create from file
kubectl create -f pod-definition.yaml
# or
kubectl apply -f pod-definition.yaml
```

Normal pod startup progression: `Pending` → `ContainerCreating` → `Running`

---

> [!tip] CKA Exam
> - Scaling = more **pods**, never more containers in the same pod
> - `kubectl run nginx --image=nginx` is the fastest way to spin up a pod imperatively
> - Use `kubectl get pods -o wide` to see which node a pod is on
> - Use `kubectl describe pod <name>` to debug — check the **Events** section at the bottom
> - Know the 4 required top-level fields in any YAML manifest: `apiVersion`, `kind`, `metadata`, `spec`
