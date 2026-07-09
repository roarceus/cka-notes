# Service Accounts

## What They Are

Service Accounts are Kubernetes-native identities for **pods and applications** (not humans). Used by apps like Prometheus, Jenkins, or custom dashboards to authenticate against the Kubernetes API.

---

## Creating and Using Service Accounts

```bash
# Create
kubectl create serviceaccount dashboard-sa

# List
kubectl get serviceaccounts

# Describe (see associated tokens)
kubectl describe serviceaccount dashboard-sa
```

### Assign to a Pod

```yaml
spec:
  serviceAccountName: dashboard-sa   # defaults to "default" if omitted
  containers:
  - name: my-app
    image: my-app
```

> You **cannot change** the service account of a running pod — delete and recreate, or update the Deployment template (triggers a rollout).

### Disable automatic token mounting

```yaml
spec:
  automountServiceAccountToken: false
```

---

## Token Behaviour by Version

| Version | Behaviour |
|---|---|
| < v1.22 | Token auto-created as a Secret — no expiry, no audience binding |
| v1.22+ | Tokens generated via **TokenRequest API** — time-bound, audience-bound, object-bound |
| v1.24+ | Kubernetes no **longer auto-creates** token Secrets — must generate explicitly |

### Generate a token (v1.24+)

```bash
# Generate a short-lived token (default 1 hour)
kubectl create token dashboard-sa

# Use the token
curl https://<api-server>:6443/api -k \
  --header "Authorization: Bearer <token>"
```

### Create a non-expiring token Secret (legacy / when needed)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-sa-token
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
type: kubernetes.io/service-account-token
```

> Avoid non-expiring tokens in production — use `kubectl create token` instead.

---

## Token Auto-Mount in Pods

The token is automatically mounted at:
```
/var/run/secrets/kubernetes.io/serviceaccount/token
```

```bash
# View the token from inside a pod
kubectl exec -it <pod> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

---

## Granting Permissions to a Service Account

Service accounts authenticate but need RBAC to be authorised:

```bash
kubectl create rolebinding dashboard-binding \
  --role=developer \
  --serviceaccount=default:dashboard-sa
```

```yaml
# In RoleBinding subjects:
subjects:
- kind: ServiceAccount
  name: dashboard-sa
  namespace: default
```

---

> [!tip] CKA Exam
> - Every namespace has a `default` service account — pods use it automatically if none is specified
> - The default SA has very limited permissions by default
> - `kubectl create token <sa-name>` generates a token — use this in v1.24+ clusters
> - To give a pod API access: create SA → create Role → create RoleBinding (referencing `kind: ServiceAccount`)
> - Service account tokens mount at `/var/run/secrets/kubernetes.io/serviceaccount/`
> - Set `automountServiceAccountToken: false` when a pod doesn't need API access (security best practice)
