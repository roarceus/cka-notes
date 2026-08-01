# Kustomize: Common Transformers

## What Transformers Do

- Transformers apply consistent changes across **all** resources managed by a `kustomization.yaml`, without editing each manifest by hand — e.g. adding a label, renaming resources, setting a namespace
- Defined as top-level fields in `kustomization.yaml`

---

## 1. Common Labels

```yaml
commonLabels:
  org: KodeKloud
```
- Adds the label to every resource's `metadata.labels` (and to relevant selector/template labels where applicable):
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    org: KodeKloud
  name: api-service
spec:
  selector:
    component: api
  ...
```

---

## 2. Namespace

```yaml
namespace: lab
```
- Sets `metadata.namespace` on every managed resource to `lab`

> [!tip] CKA Exam
> The target namespace must already exist in the cluster, or the deployment will fail.

---

## 3. Name Prefix / Suffix

```yaml
namePrefix: KodeKloud-
nameSuffix: -dev
```
- Renames every resource, e.g. `api-service` → `KodeKloud-api-service-dev`
- Useful for distinguishing environments (dev/staging/prod) without duplicating manifests

---

## 4. Common Annotations

```yaml
commonAnnotations:
  branch: master
```
- Adds the annotation to every resource's `metadata.annotations`

---

## Summary

| Transformer | Field | Effect |
|---|---|---|
| Common Labels | `commonLabels` | Adds labels to all resources |
| Namespace | `namespace` | Assigns all resources to a specific namespace |
| Name Prefix/Suffix | `namePrefix` / `nameSuffix` | Renames all resources |
| Common Annotations | `commonAnnotations` | Adds annotations to all resources |

---

> [!tip] CKA Exam
> - These four (`commonLabels`, `namespace`, `namePrefix`/`nameSuffix`, `commonAnnotations`) are top-level fields directly in `kustomization.yaml` — no separate transformer config file needed
> - `commonLabels` affects both `metadata.labels` and matching selectors/template labels — not just a cosmetic tag
> - Namespace transformer requires the namespace to **already exist** — Kustomize doesn't create it for you
