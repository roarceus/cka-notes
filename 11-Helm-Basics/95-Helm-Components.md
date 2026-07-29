# Helm Components

## Core Concepts

- **Helm CLI**: the command-line tool installed locally — used to install, upgrade, roll back releases
- **Chart**: a collection of files with instructions for creating a set of Kubernetes objects — the deployable package
- **Release**: a specific installation of a chart into a cluster
- **Revision**: each change to a release (upgrade, rollback, config change) creates a new revision, so history is tracked
- **Repository**: a place charts are published/hosted (similar to Docker Hub for images) — lets you quickly pull and deploy existing charts instead of writing them from scratch

![Flow from chart repository → Helm CLI → releases/revisions in Kubernetes](https://kodekloud.com/kk-media/image/upload/v1752869780/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Helm-Components/helm-components-chart-repository-diagram.jpg)

- Helm stores all its metadata (installed releases, charts used, revision history) **as Secrets inside the cluster itself** — persistent and shared across the team, not stored locally on one machine

---

## Charts and Templating

- A chart's manifests use **templated values** instead of hardcoded ones — actual values come from a separate `values.yaml`

Example — HelloWorld chart:
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: hello-world
```

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: nginx
          image: {{ .Values.image.repository }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

```yaml
# values.yaml
replicaCount: 1
image:
  repository: nginx
```

- Customizing a deployment is usually just editing `values.yaml` — not touching the templates themselves
- Real-world charts (e.g. Bitnami-style) use much more advanced templating (`include`, `nindent`, conditionals) to handle labels, annotations, and reusable snippets across many resources — worth recognizing the syntax, not memorizing it

---

## Releases: Multiple Installs of the Same Chart

```bash
# helm install [release-name] [chart]
helm install my-site bitnami/wordpress
helm install my-SECOND-site bitnami/wordpress
```
- Each release is tracked **independently**, even from the same chart — useful for running separate prod/dev instances, where you can experiment safely in one before promoting changes to the other

---

## Repositories and Artifact Hub

- Charts for common apps (Redis, Prometheus, etc.) are hosted across many public repositories (Bitnami, TrueCharts, Community Operators, Appscode, etc.)
- **Artifact Hub** (artifacthub.io) centralizes search across these repositories instead of checking each one individually, and flags official/verified publishers

![Helm repositories (Appscode, Community Operators, TrueCharts, Bitnami) feeding into Artifact Hub](https://kodekloud.com/kk-media/image/upload/v1752869781/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Helm-Components/helm-repositories-artifacthub-diagram.jpg)

---

> [!tip] CKA Exam
> - Know the hierarchy: **Repository → Chart → Release → Revision**
> - Helm's metadata (release/revision history) is stored **as Kubernetes Secrets** in-cluster, not client-side
> - `values.yaml` is the single point of customization for a chart — templates reference it via `.Values.*`
> - Multiple `helm install` calls with different release names can deploy the **same chart** multiple times independently
