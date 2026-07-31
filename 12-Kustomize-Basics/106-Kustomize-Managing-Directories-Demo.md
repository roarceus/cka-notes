# Kustomize: Managing Directories Demo

This walks through the same directory-per-component pattern from the previous note, applied to a concrete example (API + Redis cache + MongoDB, each in its own subdirectory).

## Deploying Without Kustomize (Baseline)

Applying multiple directories individually:
```bash
kubectl apply -f k8s/api
kubectl apply -f k8s/cache
kubectl apply -f k8s/db
```

`kubectl` also accepts **multiple `-f` flags in a single command**, for both apply and delete:
```bash
kubectl apply -f k8s/api -f k8s/cache -f k8s/db
kubectl delete -f k8s/db -f k8s/cache -f k8s/api
```

---

## With Kustomize: Single Root kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - cache/redis-config.yaml
  - cache/redis-depl.yaml
  - cache/redis-service.yaml
  - db/db-config.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
```

```bash
kustomize build k8s/          # render only, doesn't deploy
kustomize build k8s/ | kubectl apply -f -
# or
kubectl apply -k k8s/
```

Verify:
```bash
kubectl get pods
```

---

## With Kustomize: Per-Subdirectory kustomization.yaml (Scalable Pattern)

Each component gets its own `kustomization.yaml` (e.g. `cache/kustomization.yaml` listing only `redis-config.yaml`, `redis-depl.yaml`, `redis-service.yaml`), and the root just references the directories:

```yaml
# root kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - api/
  - cache/
  - db/
```

- Kustomize automatically looks for a `kustomization.yaml` **inside** any directory listed as a resource
- Deploy the same way: `kubectl apply -k k8s/`
- Expected output creates all ConfigMaps, Services, and Deployments across all three components in one command, verifiable again with `kubectl get pods`

---

> [!tip] CKA Exam
> - `kubectl apply`/`kubectl delete` accept **multiple `-f` flags** in one command — useful shorthand even without Kustomize
> - When a directory (not a file) is listed under `resources:`, Kustomize expects a `kustomization.yaml` inside that directory
> - The deployment command (`kubectl apply -k k8s/`) doesn't change whether you use a flat resource list or nested per-directory kustomization files — only the internal organization differs
