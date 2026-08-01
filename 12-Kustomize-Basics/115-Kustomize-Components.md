# Kustomize: Components

## The Problem

- **Base** = config shared by *every* overlay
- But some features are only needed by *some* overlays (e.g. caching only for Premium + Self-Hosted, external DB only for Dev + Premium)
- Putting such a feature in the base would activate it everywhere; copying it into each relevant overlay individually risks drift when it's later updated
- **Components** solve this: define the feature's config once, and let only the overlays that need it import it

![Hierarchy showing base, overlays (dev/Premium/Self-hosted), and which use caching vs external DB](https://kodekloud.com/kk-media/image/upload/v1752869797/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Components/hierarchical-flowchart-components-caching.jpg)

---

## Project Structure

```
k8s/
├── base/
│   ├── kustomization.yaml
│   └── api-depl.yaml
├── components/
│   ├── caching/
│   │   ├── kustomization.yaml
│   │   ├── deployment-patch.yaml
│   │   └── redis-depl.yaml
│   └── db/
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       └── postgres-depl.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── premium/
        └── kustomization.yaml
```
- Each component gets its own folder under `components/`, with its own resources, patches, and `kustomization.yaml`

---

## Defining a Component (Database Example)

`postgres-depl.yaml` — the resource this component provides:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: postgres
  template:
    metadata:
      labels:
        component: postgres
    spec:
      containers:
        - name: postgres
          image: postgres
```

The component's **`kustomization.yaml`** — note `kind: Component`, not `Kustomization`:
```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
resources:
  - postgres-depl.yaml
secretGenerator:
  - name: postgres-cred
    literals:
      - password=postgres123
patches:
  - deployment-patch.yaml
```
- Imports its own resource (`postgres-depl.yaml`)
- Generates a Secret (`postgres-cred`) holding the DB password
- Applies a patch to modify the *base's* existing deployment

`deployment-patch.yaml` — a strategic merge patch that wires the base API deployment to use the new secret:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-cred
                  key: password
```

---

## Using a Component in an Overlay

```yaml
# overlays/dev/kustomization.yaml
bases:
  - ../../base
components:
  - ../../components/db
```
- `bases` imports the shared base first
- `components` then layers in the optional feature — only overlays that list a given component get it

Any overlay needing the same feature (e.g. `premium`) just adds the same `components:` entry — the component's config is written once and reused consistently.

---

> [!tip] CKA Exam
> - Components use **`kind: Component`** (not `Kustomization`) and `apiVersion: kustomize.config.k8s.io/v1alpha1` in their own `kustomization.yaml`
> - Components can bundle their own resources, generators (e.g. `secretGenerator`), and patches to the base — all in one importable unit
> - An overlay opts into a component via the `components:` field, separate from `bases:`
> - Use components for **optional, cross-cutting features** that only some overlays need — avoids duplicating that config across multiple overlays
