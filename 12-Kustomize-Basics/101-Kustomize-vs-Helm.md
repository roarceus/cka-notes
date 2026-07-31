# Kustomize vs Helm

## Helm's Templating Approach

- Helm uses **Go templating syntax** (`{{ }}`) directly inside manifests, with values externalized to `values.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "nginx:{{ .Values.image.tag }}"
```
```yaml
# values.yaml
replicaCount: 1
image:
  tag: "2.4.4"
```
- At deploy time, Helm merges the values file into the template, replacing placeholders with real values

### Typical Helm Project Layout
```
k8s/
  environments/
    values.dev.yaml
    values.stg.yaml
    values.prod.yaml
  templates/
    nginx-deployment.yaml
    nginx-service.yaml
    db-deployment.yaml
    db-service.yaml
```
- The right environment's values file is selected at deploy time to inject the correct settings into the shared templates

---

## Helm's Trade-offs

- Helm is a full **package manager** for Kubernetes apps (like `apt`/`yum` for Linux) — beyond just environment templating, it also supports:
  - Conditionals and loops
  - Functions and hooks
- This power comes at a cost: Helm templates use Go templating syntax, so they are **not valid YAML on their own** — harder to read/maintain, especially as charts grow complex

---

## Kustomize vs Helm — Core Difference

| | Helm | Kustomize |
|---|---|---|
| Config format | Go templates (not plain YAML) | Plain, standard YAML |
| Customization model | `values.yaml` + template placeholders | Base + overlays |
| Extra features | Package management, conditionals, loops, functions, hooks | None — intentionally minimal |
| Learning curve | Higher (templating language to learn) | Lower (just YAML) |
| Readability | Can get hard to read in complex charts | Stays readable — no special syntax |

---

## Choosing Between Them

- Choose **Helm** if you need advanced functionality: packaging/distribution, conditionals/loops, hooks, or dynamic templating logic
- Choose **Kustomize** if you want a simpler, plain-YAML approach with less to learn and easier long-term readability

---

> [!tip] CKA Exam
> - Helm = **templating engine + package manager** (Go templates, `values.yaml`, charts/releases/revisions)
> - Kustomize = **plain YAML + base/overlay model**, no templating language at all
> - Know this distinction is the primary reason to pick one over the other — Kustomize trades power for simplicity, Helm trades simplicity for power
