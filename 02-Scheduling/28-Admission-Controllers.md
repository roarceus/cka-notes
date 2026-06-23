# Admission Controllers

## Where They Fit in the Request Flow

```
kubectl request
    → Authentication (certificates / kubeconfig)
    → Authorization (RBAC)
    → Admission Controllers   ← here
    → etcd (persisted)
```

Admission controllers intercept requests **after** auth/authz but **before** the object is saved. They can:
- **Validate** — reject requests that don't meet policy
- **Mutate** — modify the request before it's persisted (e.g. inject defaults)

---

## Why RBAC Alone Isn't Enough

RBAC works at the API level — it can allow or deny operations on resources. It **cannot** inspect or change the object's content.

Examples of what admission controllers can enforce that RBAC can't:
- Reject pods using images from public registries
- Reject pods running as root
- Auto-inject labels or sidecars
- Auto-create namespaces

---

## Common Built-in Admission Controllers

| Controller | What it does |
|---|---|
| `AlwaysPullImages` | Forces image pull on every pod start |
| `DefaultStorageClass` | Injects a default storage class into PVCs that don't specify one |
| `EventRateLimit` | Throttles API server request rate |
| `NamespaceLifecycle` | Rejects requests to non-existent namespaces; protects `default`, `kube-system`, `kube-public` from deletion |
| `NodeRestriction` | Limits what kubelets can modify |
| `NamespaceAutoProvision` | ⚠️ Deprecated — auto-creates namespace if missing |

> `NamespaceAutoProvision` and `NamespaceExists` are **deprecated** — replaced by `NamespaceLifecycle`.

---

## Enabling / Disabling Admission Controllers

### View currently enabled plugins
```bash
kube-apiserver -h | grep enable-admission-plugins

# On kubeadm clusters (API server runs as a pod):
kubectl exec -n kube-system kube-apiserver-controlplane -- kube-apiserver -h | grep enable-admission-plugins
```

### Enable via kube-apiserver flag
```bash
# In /etc/kubernetes/manifests/kube-apiserver.yaml (kubeadm clusters):
- --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision

# Disable:
- --disable-admission-plugins=DefaultStorageClass
```

After editing the manifest, the API server pod restarts automatically.

---

---

## Mutating vs Validating Admission Controllers

| Type | What it does | Runs |
|---|---|---|
| **Mutating** | Modifies the object before persisting (e.g. injects defaults, adds labels) | First |
| **Validating** | Validates the object and allows or rejects it | After mutating |

> Mutating runs first so that validating controllers see the fully modified object. If any controller rejects the request, the whole request is denied.

![Mutating and Validating controller flow](https://kodekloud.com/kk-media/image/upload/v1752869883/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-2025-Updates-Validating-and-Mutating-Admission-Controllers/frame_140.jpg)

---

## Webhook Admission Controllers

For custom logic beyond built-in controllers, Kubernetes supports external **admission webhooks**. The API server sends an `AdmissionReview` JSON object to your webhook server, which responds with allow/deny.

Two webhook types:
- `MutatingWebhookConfiguration`
- `ValidatingWebhookConfiguration`

### ValidatingWebhookConfiguration example

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-policy.example.com
webhooks:
- name: pod-policy.example.com
  clientConfig:
    service:
      namespace: webhook-namespace
      name: webhook-service    # webhook server running inside the cluster
    caBundle: "Ci0tLS0tQk..."  # TLS cert bundle
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]     # triggers on pod creation
    resources: ["pods"]
    scope: "Namespaced"
```

The webhook server receives an `AdmissionReview` request and returns `"allowed": true/false`.

---

> [!tip] CKA Exam
> - Admission controllers sit **after** RBAC in the request chain — they can inspect and modify object content
> - Enable/disable via `--enable-admission-plugins` / `--disable-admission-plugins` flags on the API server
> - On kubeadm clusters, edit `/etc/kubernetes/manifests/kube-apiserver.yaml` to change these flags
> - `NamespaceLifecycle` is the active replacement for the deprecated namespace controllers
> - If a pod is rejected with a policy-related error (not a permissions error), an admission controller is likely the cause
