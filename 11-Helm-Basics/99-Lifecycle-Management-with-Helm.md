# Lifecycle Management with Helm

## Installing a Specific Version

```bash
helm install nginx-release bitnami/nginx --version 7.1.0
```
- `--version` pins the chart version at install time (useful for reproducing an older/known-good state)
- Verify the actual deployed image:
```bash
kubectl describe pod nginx-release-687cdd5c75-ztn2n
# Image: docker.io/bitnami/nginx:1.19.2-debian-10-r28
```

---

## Upgrading a Release

```bash
helm upgrade nginx-release bitnami/nginx
```
- Replaces the old pod(s) with new ones reflecting the updated chart/image/config
- Creates a **new revision** (e.g. revision 2)

Check current state and history:
```bash
helm list
# NAME           NAMESPACE  REVISION  STATUS    CHART          APP VERSION
# nginx-release  default    2         deployed  nginx-9.5.13   1.21.4

helm history nginx-release
# REVISION  STATUS      CHART          APP VERSION  DESCRIPTION
# 1         superseded  nginx-7.1.0    1.19.2       Install complete
# 2         deployed    nginx-9.5.13   1.21.4       Upgrade complete
```

---

## Rolling Back

```bash
helm rollback nginx-release 1
```
- Reverts the release to revision 1's configuration
- Helm records the rollback itself as a **new revision** (e.g. revision 3) — full history is preserved for auditing

> [!tip] CKA Exam
> Rollbacks revert the Kubernetes manifest configuration only — they do **not** restore data in PersistentVolumes or external databases. A separate backup/restore strategy is still needed for actual data.

---

## Upgrades on Complex Charts (Credential Requirements)

Some charts (e.g. WordPress + MariaDB) require certain sensitive values to be re-supplied on every upgrade — omitting them fails the upgrade:

```bash
helm upgrade wordpress-release bitnami/wordpress
# Error: 'wordpressPassword' must not be empty, please add
# '--set wordpressPassword=$WORDPRESS_PASSWORD' to the command
```

- The error message itself tells you how to retrieve the current value from the existing Secret, e.g.:
```bash
export WORDPRESS_PASSWORD=$(kubectl get secret wordpress-release -o jsonpath="{.data.wordpress-password}" | base64 --decode)
helm upgrade wordpress-release bitnami/wordpress --set wordpressPassword=$WORDPRESS_PASSWORD
```
- This happens because credentials (passwords) aren't stored in `values.yaml` for security — they must be explicitly passed again unless retrieved from the existing Secret

---

## Summary

| Action | Purpose | Example |
|---|---|---|
| Install | Create a new release | `helm install my-release bitnami/nginx --version 7.1.0` |
| Upgrade | Update an existing release | `helm upgrade my-release bitnami/nginx` |
| Rollback | Revert to a previous revision | `helm rollback my-release 1` |

---

> [!tip] CKA Exam
> - Every install/upgrade/rollback creates a new **revision** — `helm history <release>` shows the full trail
> - Rollback does **not** restore persistent volume or database data — only the Kubernetes object configs
> - Upgrading charts with required secrets (passwords, etc.) often needs those values re-passed via `--set`, since they aren't kept in `values.yaml`
> - `--version` on `helm install`/`helm upgrade` pins to a specific chart version
