# Helm Charts

## Chart.yaml Metadata

Every chart has a `Chart.yaml` with metadata about the chart itself (separate from `values.yaml`, which holds configurable values):

| Field | Purpose |
|---|---|
| `apiVersion` | `v2` for Helm 3 charts (Helm 2 charts use `v1` or omit it) |
| `appVersion` | Version of the **application** being deployed (e.g. WordPress version) |
| `version` | Version of the **chart** itself — independent of the app version |
| `name` / `description` | Identification and summary |
| `type` | `application` (default) or `library` |
| `dependencies` | Other charts this chart relies on |
| Optional: `keywords`, `maintainers`, `home`, `icon` | Discovery/branding metadata |

Example — WordPress chart with a MariaDB dependency:
```yaml
apiVersion: v2
appVersion: 5.8.1
version: 12.1.27
name: wordpress
description: Web publishing platform for building blogs and websites.
type: application
dependencies:
  - condition: mariadb.enabled
    name: mariadb
    repository: https://charts.bitnami.com/bitnami
    version: 9.x.x
keywords:
  - application
  - blog
  - wordpress
maintainers:
  - email: containers@bitnami.com
    name: Bitnami
home: https://github.com/bitnami/charts/tree/master/bitnami/wordpress
icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png
```
- `dependencies` entries can be conditional (`condition: mariadb.enabled`) — the dependency only installs if that value is set true

---

## Chart Directory Structure

```
mychart/
├── Chart.yaml       # chart metadata
├── values.yaml       # default configuration values
├── templates/        # Kubernetes manifest templates (Deployment, Service, etc.)
├── charts/           # optional: bundled dependent charts (e.g. MariaDB for WordPress)
├── LICENSE           # optional
└── README            # optional
```

---

## Installing from a Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/wordpress
```
- `helm repo add` registers a chart repository locally
- `helm install <release-name> <repo>/<chart>` installs a chart from that repo as a new release

---

> [!tip] CKA Exam
> - `Chart.yaml`'s `version` (chart version) vs `appVersion` (app version) are tracked **independently** — don't confuse them
> - `apiVersion: v2` in `Chart.yaml` signals a Helm 3 chart
> - Dependencies declared in `Chart.yaml` can be conditionally enabled (e.g. `mariadb.enabled`)
> - Standard chart layout: `Chart.yaml`, `values.yaml`, `templates/`, optional `charts/` for sub-dependencies
