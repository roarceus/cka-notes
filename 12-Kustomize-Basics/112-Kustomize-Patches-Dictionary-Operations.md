# Kustomize Patches: Dictionary Operations (Update / Add / Remove)

Base Deployment used throughout (label `component: api`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        component: api
```

## Updating an Existing Key

**JSON 6902** — `replace` at the exact path:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/metadata/labels/component
        value: web
```

**Strategic merge** — just restate the key with its new value:
```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        component: web
```
```yaml
# kustomization.yaml
patches:
  - label-patch.yaml
```

---

## Adding a New Key

**JSON 6902** — `add` at the new path:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        path: /spec/template/metadata/labels/org
        value: KodeKloud
```

**Strategic merge** — include just the new key; existing keys (e.g. `component: api`) are preserved automatically:
```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: KodeKloud
```
- Result: both `component: api` and `org: KodeKloud` exist on the resource

---

## Removing a Key

**JSON 6902** — `remove` (no `value` needed):
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org
```

**Strategic merge** — set the key to `null`:
```yaml
# label-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: null
```
- Kustomize interprets `null` as "delete this key" rather than literally setting it to null

---

> [!tip] CKA Exam
> - JSON 6902 dictionary paths go all the way down to the specific key: `/spec/template/metadata/labels/<key>`
> - `add` and `replace` both need a `value`; `remove` does not
> - Strategic merge patch: adding/updating a key = just include it with the desired value (untouched keys are preserved); **removing** a key = set its value to `null`
> - Double-check the exact path/key before removing — an incorrect path risks deleting or missing the wrong field
