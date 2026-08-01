# Kustomize Patches: Inline vs Separate File

Both JSON 6902 and strategic merge patches can be defined either **inline** (embedded directly in `kustomization.yaml`) or in a **separate file** (referenced by path). The choice doesn't depend on patch type — it's about how many patches you have and how clean you want `kustomization.yaml` to stay.

## Inline

Best for a few simple changes:
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |
      - op: replace
        path: /spec/replicas
        value: 5
```

---

## Separate File

Better when managing multiple/complex patches — keeps `kustomization.yaml` clean:

```yaml
# kustomization.yaml
patches:
  - path: replica-patch.yaml
    target:
      kind: Deployment
      name: nginx-deployment
```

```yaml
# replica-patch.yaml
- op: replace
  path: /spec/replicas
  value: 5
```
- `path` points to the external patch file; `target` still specifies which object(s) to apply it to

---

## Strategic Merge Patch as a Full Resource File

For strategic merge patches, the external file can be a **complete partial resource** (no `op`/`path` needed — matching happens via `apiVersion`/`kind`/`metadata.name`):

```yaml
# kustomization.yaml
patches:
  - path: replica-patch.yaml
```

```yaml
# replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 5
```
- Note: for this style, no separate `target` block is needed — the patch file's own `metadata.name`/`kind` identifies what it applies to

---

> [!tip] CKA Exam
> - Inline vs separate-file is an **organizational** choice, not tied to JSON 6902 vs strategic merge — both patch types support both styles
> - JSON 6902 patches always need an explicit `target` (kind/name/etc.) since the patch body itself (`op`/`path`/`value`) has no identifying info
> - Strategic merge patches as a full resource file are self-identifying (via their own `apiVersion`/`kind`/`metadata.name`) and don't require a separate `target` block
