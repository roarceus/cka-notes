# Kustomize Patches: List Operations (Containers)

Base Deployment — `containers` is a **list**, so operations reference it by **index** (JSON 6902) or by **name** (strategic merge):
```yaml
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: nginx
```

## Replacing a List Item

**JSON 6902** — replace by index (indices are zero-based):
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0
        value:
          name: haproxy
          image: haproxy
```
- Replaces the *entire* container at index 0 (both name and image)

**Strategic merge** — matches by container `name`, only overriding the fields you specify:
```yaml
# label-patch.yaml
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: haproxy
```
- Finds the container named `nginx` and updates just its `image`, leaving other fields alone

---

## Adding a List Item

**JSON 6902** — `/containers/-` appends to the **end** of the list:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        path: /spec/template/spec/containers/-
        value:
          name: haproxy
          image: haproxy
```
- Use a specific numeric index instead of `-` if you need the new item inserted at a particular position

**Strategic merge** — just list the additional container; existing ones are preserved:
```yaml
# label-patch.yaml
spec:
  template:
    spec:
      containers:
        - name: haproxy
          image: haproxy
```

---

## Removing a List Item

**JSON 6902** — `remove` by index:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: remove
        path: /spec/template/spec/containers/1
```
- Removes whatever container currently sits at index 1 — double-check the index first

**Strategic merge** — use the special `$patch: delete` directive, matched by name:
```yaml
# label-patch.yaml
spec:
  template:
    spec:
      containers:
        - $patch: delete
          name: database
```

---

> [!tip] CKA Exam
> - JSON 6902 list paths use a **numeric index** (`/containers/0`, `/containers/1`, ...); `/containers/-` specifically means "append to the end"
> - Strategic merge patches match list items **by name**, not index — much less fragile if the list order changes
> - Strategic merge delete uses `$patch: delete` alongside the matching `name` — not a `null` value like dictionary key removal
> - Always double check indices before a JSON 6902 `remove`/`replace` on a list — an off-by-one hits the wrong container
