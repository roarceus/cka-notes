# Customizing Helm Chart Parameters

## Where Default Values Come From

- A chart's `values.yaml` defines defaults that templates reference via `.Values.*`
- Example (Bitnami WordPress) — `values.yaml` sets `wordpressBlogName: User's Blog!`, and the Deployment template consumes it as an env var:
```yaml
env:
  - name: WORDPRESS_BLOG_NAME
    value: {{ .Values.wordpressBlogName | quote }}
```
- Unless overridden, the app deploys with whatever `values.yaml` specifies

---

## Method 1: `--set` on the CLI

Best for a few quick overrides:
```bash
helm install --set wordpressBlogName="Helm Tut" my-release bitnami/wordpress
```
- Multiple values can be set at once (comma-separated with `--set`)
- CLI `--set` values **take precedence** over `values.yaml` defaults

---

## Method 2: Custom Values File

Better for many overrides — create your own values file and pass it in:

```yaml
# custom-values.yaml
wordpressBlogName: Helm Tut
wordpressEmail: john@example.com
```

```bash
helm install my-release bitnami/wordpress --values custom-values.yaml
```
- `--values`/`-f` merges your file's values over the chart's defaults

---

## Method 3: Editing the Chart's Built-in `values.yaml`

For deeper/persistent changes to the chart itself:

```bash
helm pull bitnami/wordpress               # download chart archive
helm pull --untar bitnami/wordpress       # download and extract to a directory
```

```bash
ls wordpress
# Chart.yaml  Chart.lock  README.md  values.yaml  values.schema.json  templates  ci
```

- Edit `wordpress/values.yaml` directly, then install from the local path:
```bash
helm install my-release ./wordpress
```
- `./` tells Helm to use the local chart directory instead of pulling from a repo

---

> [!tip] CKA Exam
> - Precedence: `--set` > custom values file (`--values`) > chart's built-in `values.yaml` defaults
> - `helm pull --untar <chart>` downloads and extracts a chart locally for direct editing
> - Installing from a local path uses `helm install <release-name> ./<chart-dir>` (note the `./`)
> - Know all three customization methods and when each makes sense: quick tweak (`--set`), many overrides (custom values file), or structural/persistent changes (edit and install locally)
