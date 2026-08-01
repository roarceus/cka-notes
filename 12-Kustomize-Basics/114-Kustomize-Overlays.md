# Kustomize: Overlays

## The Model

- **Base**: shared/default Kubernetes resources, common across all environments
- **Overlay**: one per environment (dev, stg, prod), each with its own `kustomization.yaml` that references the base and layers on environment-specific patches/resources

![Base directory with shared configs, overlay directories per environment](https://kodekloud.com/kk-media/image/upload/v1752869807/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Overlays/kubernetes-config-directory-structure.jpg)

Typical layout:
```
.
├── base/
│   ├── kustomization.yaml
│   ├── nginx-depl.yaml
│   ├── service.yaml
│   └── redis-depl.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── stg/
    │   └── kustomization.yaml
    └── prod/
        └── kustomization.yaml
```

---

## Base Configuration

```yaml
# base/kustomization.yaml
resources:
  - nginx-depl.yaml
  - service.yaml
  - redis-depl.yaml
```

```yaml
# base/nginx-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
```

---

## Overlay: Referencing the Base + Patching

```yaml
# overlays/dev/kustomization.yaml
bases:
  - ../../base
patch: |-
  - op: replace
    path: /spec/replicas
    value: 2
```
- `bases` points to the base directory (relative path)
- The patch overrides `replicas` to 2 for dev

```yaml
# overlays/prod/kustomization.yaml
bases:
  - ../../base
patch: |-
  - op: replace
    path: /spec/replicas
    value: 3
```
- Same base, different replica count for production

---

## Overlay: Adding Environment-Exclusive Resources

An overlay can also introduce resources that don't exist in the base at all:

```yaml
# overlays/prod/kustomization.yaml
bases:
  - ../../base
resources:
  - grafana-depl.yaml
patch: |-
  - op: replace
    path: /spec/replicas
    value: 2
```
- Imports the base, adds a prod-only Grafana deployment, and patches the replica count — all in one overlay

---

## Structural Flexibility

- The base can be organized into feature subdirectories, but overlay directories **don't need to mirror that structure** — what matters is that each overlay correctly references the shared base resources

---

> [!tip] CKA Exam
> - `bases` in an overlay's `kustomization.yaml` points to the shared base directory (often via a relative path like `../../base`)
> - Overlays can both **patch existing base resources** and **add new, environment-exclusive resources** in the same `kustomization.yaml`
> - The base/overlay model keeps shared config in one place while letting each environment diverge only where needed — no duplication of the full manifest set per environment
