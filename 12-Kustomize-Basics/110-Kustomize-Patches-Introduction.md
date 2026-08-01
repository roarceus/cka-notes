# Kustomize: Patches — Introduction

## Transformers vs Patches

- **Transformers** (`commonLabels`, `namespace`, etc.) apply broad, global changes across all managed resources
- **Patches** are surgical — they target one (or a few) specific object(s) for precise modification, e.g. changing the replica count on just one particular Deployment

---

## Anatomy of a Patch

Every patch needs three things:

1. **Operation** — the action to perform:
   - `add` — insert a new element (e.g. a new container)
   - `remove` — delete an element (e.g. a container or label) — no value needed
   - `replace` — swap an existing value for a new one (e.g. replicas 1 → 5)
2. **Target** — criteria to select which object(s) to patch: `kind`, `version`, `name`, `namespace`, label selector, annotation selector (can be combined to narrow the match)
3. **Value** — the new value to apply (omitted for `remove`)

---

## Example: JSON 6902 Patch — Rename a Deployment

Base:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  ...
```

In `kustomization.yaml`:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
```
- Result: `metadata.name` becomes `web-deployment`

---

## Example: JSON 6902 Patch — Change Replica Count

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```
- Result: `spec.replicas` becomes `5`

---

## Two Patch Styles

### JSON 6902 Patch
- Specifies a `target` plus a list of `op`/`path`/`value` operations (path-based, precise)
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch:
      - op: replace
        path: /spec/replicas
        value: 5
```

### Strategic Merge Patch
- Looks like a normal (partial) Kubernetes manifest — only include the fields you want to change, and Kustomize merges them into the existing resource
```yaml
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5
```
- No explicit `op`/`path` needed — matching is implicit via `apiVersion`/`kind`/`metadata.name`

Both achieve the same result — choice is largely preference, though many find strategic merge patches more readable since they resemble ordinary YAML manifests.

---

> [!tip] CKA Exam
> - Patches = targeted, single-object changes; transformers = broad, all-resource changes
> - JSON 6902 patch = explicit `op`/`path`/`value` operations against a `target`
> - Strategic merge patch = a partial manifest that gets merged in — no `op`/`path` syntax
> - `remove` operations don't need a `value` — only `add` and `replace` do
