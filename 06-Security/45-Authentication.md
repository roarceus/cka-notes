# Authentication

## Key Concept — Users vs Service Accounts

| | Regular Users | Service Accounts |
|---|---|---|
| Kubernetes object? | ❌ No — not managed by Kubernetes | ✅ Yes — created via API |
| Used by | Humans (admins, devs), external tools | Pods and in-cluster processes |
| Created with | External mechanism (certs, OIDC) | `kubectl create serviceaccount` |

```bash
kubectl create serviceaccount sa1
kubectl get serviceaccounts
```

All requests to the kube-apiserver — from kubectl, dashboards, controllers, or direct API calls — are authenticated before authorization is applied.

---

## Authentication Mechanisms

| Mechanism | Use case | Recommended? |
|---|---|---|
| Static basic-auth file | Legacy/labs only | ❌ Deprecated |
| Static token file | Legacy/labs only | ❌ Deprecated |
| **TLS client certificates** | Admin/user access, machine access | ✅ Yes |
| **External providers (OIDC, LDAP)** | Enterprise SSO, multi-tenant clusters | ✅ Yes |

---

## Legacy: Static Files (know for context, don't use in production)

### Static basic-auth file
CSV format: `password,username,uid,groups`

```bash
# Enable on kube-apiserver (deprecated)
--basic-auth-file=/path/to/basic-auth.csv

# Authenticate
curl -k -u user10:password123 https://<master-ip>:6443/api
```

### Static token file
CSV format: `token,username,uid,groups`

```bash
# Enable on kube-apiserver (deprecated)
--token-auth-file=/path/to/token-auth.csv

# Authenticate
curl -k -H "Authorization: Bearer <token>" https://<master-ip>:6443/api
```

> On kubeadm clusters, add these flags to `/etc/kubernetes/manifests/kube-apiserver.yaml` and mount the file via a `hostPath` volume. The API server restarts automatically.

---

> [!tip] CKA Exam
> - **Kubernetes does not manage regular users** — you can't `kubectl get users`
> - **Service Accounts** are the Kubernetes-native identity for pods — these you can create and manage
> - Static auth files are deprecated and insecure — understand them conceptually but the exam focuses on certificates and RBAC
> - All authentication happens at the **kube-apiserver** — it's the gateway for every API request
> - Always combine authentication with **RBAC** authorization — auth alone doesn't restrict what users can do
