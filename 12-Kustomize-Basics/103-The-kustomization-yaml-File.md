# The kustomization.yaml File

## What It Is

- Kustomize doesn't scan your Kubernetes YAML files directly — it looks for one specific file named exactly **`kustomization.yaml`**, which you create yourself in the directory alongside your manifests
- This file has two core sections:
  1. **`resources`** — a list of the Kubernetes manifest files Kustomize should manage
  2. **Customizations/transformations** — what to change/apply on top of those resources (e.g. adding labels)

Example layout:
```
k8s/
  nginx-deployment.yaml
  nginx-service.yaml
  kustomization.yaml
```

Example `kustomization.yaml`:
```yaml
resources:
  - nginx-deployment.yaml
  - nginx-service.yaml

commonLabels:
  company: KodeKloud
```
- `resources` tells Kustomize which manifests to pull in
- `commonLabels` here is one simple example of a transformation — it adds `company: KodeKloud` as a label to **every** resource Kustomize manages (many other transformation types exist beyond this)

---

## Building the Output

```bash
kustomize build k8s/
```
- Points Kustomize at the directory containing `kustomization.yaml`
- Kustomize reads the listed resources, applies the defined transformations, and **prints the final resulting YAML to the terminal**
- In the output you'd see both the Deployment and Service, each now carrying the `company: KodeKloud` label

**Important:** `kustomize build` does **not** deploy anything to the cluster — it only renders the final config. To actually apply it:
```bash
kustomize build k8s/ | kubectl apply -f -
```

---

> [!tip] CKA Exam
> - The file must be named exactly **`kustomization.yaml`** — Kustomize won't work without it
> - `kustomization.yaml` has (at minimum) a `resources` list and any transformations (e.g. `commonLabels`)
> - `kustomize build <dir>` only **renders** output — it doesn't apply anything; pipe it to `kubectl apply -f -` to actually deploy
> - Also usable natively via `kubectl apply -k <dir>` (kubectl's built-in Kustomize support), without needing the separate `kustomize build` + pipe step
