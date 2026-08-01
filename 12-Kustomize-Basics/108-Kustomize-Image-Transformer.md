# Kustomize: Image Transformer

## Purpose

- Lets you change an image name and/or tag used by containers **without editing the deployment manifest itself**
- Configured via the `images` field in `kustomization.yaml`

Base deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: web
  template:
    metadata:
      labels:
        component: web
    spec:
      containers:
        - name: web
          image: nginx
```

---

## Replacing the Image Name

```yaml
images:
  - name: nginx
    newName: haproxy
```
- Kustomize scans all managed manifests for containers using image `nginx` and replaces with `haproxy`
- Result: `image: haproxy`

> [!tip] CKA Exam
> The `name` field here matches the **image reference** (e.g. `nginx`), not the container's `name` field (e.g. `web`) — a common point of confusion.

---

## Updating Only the Tag

```yaml
images:
  - name: nginx
    newTag: 2.4
```
- Result: `image: nginx:2.4`

---

## Changing Both Name and Tag

```yaml
images:
  - name: nginx
    newName: haproxy
    newTag: 2.4
```
- Result: `image: haproxy:2.4`

---

> [!tip] CKA Exam
> - `images` transformer matches by the **image name**, not the container name
> - `newName` alone swaps the image; `newTag` alone swaps just the tag; both together change the full image reference
> - This lets you bump/change images per-environment (e.g. via an overlay) without touching the base deployment YAML
