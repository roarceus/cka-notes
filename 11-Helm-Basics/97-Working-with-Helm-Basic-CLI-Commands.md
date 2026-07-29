# Working with Helm - Basic CLI Commands

## Getting Help

```bash
helm --help
```
- Lists top-level commands and common actions (`search`, `pull`, `install`, `list`, etc.) — useful for quickly finding the right subcommand (e.g. reverting a failed upgrade is `helm rollback`, not "restore")

---

## Managing Chart Repositories

```bash
helm repo add <name> <url>      # add a chart repository
helm repo list                  # list configured repositories
helm repo update                # refresh local repo cache with latest chart info
helm repo remove <name>         # remove a repository
```

Example:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
# "bitnami" has been added to your repositories

helm repo list
# NAME    URL
# bitnami https://charts.bitnami.com/bitnami
```

---

## Searching for Charts

```bash
helm search hub wordpress    # search Artifact Hub (public, across all repos)
helm search wordpress        # search only locally added/cached repos
```
- Returns chart version, app version, and description — useful for matching a chart version to the app version you actually want

---

## Installing an Application

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/wordpress
```

Output confirms the release name, namespace, status, revision, chart/app version, and access details (e.g. an in-cluster DNS name to reach the app).

---

## Managing Releases

```bash
helm list
# NAME        NAMESPACE   REVISION  STATUS    CHART              APP VERSION
# my-release  default     1         deployed  wordpress-12.1.27  5.8.1
```

```bash
helm uninstall my-release
```
- Removes **all** Kubernetes objects created by that release — no need to track/delete objects manually

---

> [!tip] CKA Exam
> - `helm search hub` = search Artifact Hub (public); `helm search <repo>` (no `hub`) = search your **local** added repos only
> - `helm repo update` refreshes local chart metadata cache — do this before installing if repo contents may have changed
> - `helm uninstall` removes every object tied to that release automatically
> - `helm list` shows release name, namespace, revision, status, and chart/app version — the quick way to check what's currently deployed
