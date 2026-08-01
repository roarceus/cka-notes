# Kustomize: Transformers Demo — How Root and Subdirectory Transformers Combine

This builds on the per-subdirectory `kustomization.yaml` pattern (root referencing `api/` and `db/`, each with its own `kustomization.yaml`) to show how transformers **stack** across levels.

## Labels Stack Across Root and Subdirectory

- A `commonLabels` set in the **root** `kustomization.yaml` applies to **every** resource across all subdirectories
- A `commonLabels` set in a **subdirectory's own** `kustomization.yaml` applies **only to that subdirectory's resources** — and combines with the root label

Example:
```yaml
# root kustomization.yaml
commonLabels:
  department: engineering
```
```yaml
# api/kustomization.yaml
commonLabels:
  feature: api
```
```yaml
# db/kustomization.yaml
commonLabels:
  feature: db
```

Result:
- API resources get **both** `department: engineering` AND `feature: api`
- Database resources get **both** `department: engineering` AND `feature: db`

This same stacking behavior applies to `namePrefix`/`nameSuffix` too — a `namePrefix: KodeKloud-` at the root combines with a `nameSuffix: -web` set in the API subdirectory, producing names like `KodeKloud-api-deployment-web`, while the db subdirectory's `nameSuffix: -storage` produces `KodeKloud-db-deployment-storage`.

- `namespace` and `commonAnnotations` set at the root apply globally, same as `commonLabels`

---

## Image Transformer Tag Quoting Gotcha

When setting `newTag` to something that looks numeric (e.g. a version like `4.2`), **quote it as a string**:

```yaml
images:
  - name: mongo
    newName: postgres
    newTag: "4.2"
```

Without quotes, YAML parses `4.2` as a number, not a string, causing an error:
```
Error: accumulating resources: ... json: cannot unmarshal number into Go struct field Image.images.newTag of type string
```

Result (correctly quoted): `image: postgres:4.2` — only the image changes, the container's own `name` field is untouched.

---

> [!tip] CKA Exam
> - Transformers set at the **root** apply cluster-wide across all referenced subdirectories; transformers set **inside a subdirectory's own `kustomization.yaml`** apply only to that subdirectory's resources — and both levels **combine**, they don't override each other
> - Always quote `newTag` values that look numeric (e.g. `"4.2"`) to avoid a YAML type error
