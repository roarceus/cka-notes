# Kustomize: Problem Statement and Ideology

## The Problem

- A single Deployment manifest (e.g. an nginx Deployment) often needs to behave differently per environment — e.g. replica count: 1 in dev, 2–3 in staging, 5–10 in production
- **Naive solution**: duplicate the manifest into separate `dev/`, `staging/`, `production/` folders, and tweak the differing fields (e.g. `replicas`) in each copy, then:
```bash
kubectl apply -f dev/
kubectl apply -f staging/
```
- This works, but doesn't scale:
  - Every new resource (e.g. a new Service) must be manually copied into **every** environment folder
  - Every future change must be made **consistently across all folders** — easy to forget one, causing config drift between environments
  - Maintenance burden grows linearly (or worse) with the number of environments and resources

This duplication problem is the core reason Kustomize was created: **reuse configs, and only override what actually needs to differ per environment** — without copying everything.

---

## Base and Overlays

Kustomize's model has two core concepts:

- **Base**: the config that's identical (or serves as sensible defaults) across all environments — the shared source of truth
- **Overlay**: environment-specific customizations layered on top of the base — only the fields that need to change, plus any resources exclusive to that environment

Example:
- Base nginx Deployment sets `replicas: 1` (acts as the default, matching dev's needs)
- Staging overlay overrides `replicas: 2`
- Production overlay overrides `replicas: 5`
- Dev needs no overlay at all, since the base already matches what dev wants

---

## Folder Structure

```
.
├── base/
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    ├── staging/
    │   └── kustomization.yaml
    └── production/
        └── kustomization.yaml
```
- `base/` holds the shared Kubernetes resources
- `overlays/<env>/` holds only the overrides (and any env-exclusive resources) for that environment
- Kustomize combines base + the selected overlay to produce the final manifests applied to the cluster

---

## How Kustomize Differs from Helm

- Kustomize ships **built into `kubectl`** — no separate install required (though a standalone install may be needed for the latest version, since the bundled one can lag)
- **No templating language** — unlike Helm's `{{ .Values.x }}` syntax, Kustomize configs are **plain, standard YAML**
- Because everything is plain YAML (no special syntax), Kustomize output can be read, validated, and processed like any other YAML — arguably more readable than complex Helm charts full of templating logic
- Trade-off: Kustomize favors simplicity and native YAML over Helm's more powerful (but more complex) templating/packaging model

---

> [!tip] CKA Exam
> - Core Kustomize vocabulary: **base** (shared/default config) vs **overlay** (per-environment overrides)
> - Kustomize uses **plain YAML with no templating language** — this is the key conceptual difference from Helm
> - Kustomize is built into `kubectl` (`kubectl apply -k <dir>`), though a standalone binary may offer a newer version
> - The problem Kustomize solves: avoiding config duplication across environments while still allowing per-environment customization
