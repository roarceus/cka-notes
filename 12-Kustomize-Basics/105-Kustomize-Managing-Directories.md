# Kustomize: Managing Directories

## The Scaling Problem

- A flat directory with all manifests (`kubectl apply -f k8s/`) works fine for a handful of files, but gets cluttered fast
- Splitting into subdirectories by component (`k8s/api/`, `k8s/db/`) helps organize files, but now requires **separate `kubectl apply` commands per subdirectory** — tedious, especially in CI/CD pipelines

---

## Single Root kustomization.yaml Referencing Multiple Subdirectories

Instead of applying each subdirectory separately, list all resources (with their relative paths) in one root `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
```

Deploy everything in one shot:
```bash
kustomize build k8s/ | kubectl apply -f -
# or
kubectl apply -k k8s/
```

As the project grows (adding Redis, Kafka, etc.), the resource list just grows too:
```yaml
resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - cache/redis-depl.yaml
  - cache/redis-service.yaml
  - cache/redis-config.yaml
  - kafka/kafka-depl.yaml
  - kafka/kafka-service.yaml
  - kafka/kafka-config.yaml
```
- Works, but a single flat list of every file across every subdirectory becomes unwieldy at scale

---

## Better: Per-Subdirectory kustomization.yaml Files

Give each subdirectory its **own** `kustomization.yaml` listing just its local files:

```yaml
# db/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db-depl.yaml
  - db-service.yaml
```

Then the **root** `kustomization.yaml` just references the subdirectories themselves, not individual files:

```yaml
# root kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - api/
  - db/
  - cache/
  - kafka/
```

- Kustomize recursively processes each subdirectory's own `kustomization.yaml` and aggregates everything
- More modular and maintainable — each component owns its own resource list, and the root just composes them

Deployment is unchanged:
```bash
kustomize build k8s/ | kubectl apply -f -
# or
kubectl apply -k k8s/
```

---

> [!tip] CKA Exam
> - A `kustomization.yaml`'s `resources` list can point to **individual files**, or to a **directory** (which must itself contain a `kustomization.yaml`) — Kustomize resolves directories recursively
> - Structuring with per-component `kustomization.yaml` files (rather than one giant flat resource list) is the scalable pattern for larger projects
> - Deployment command stays the same regardless of how nested the structure is: `kubectl apply -k <root-dir>`
