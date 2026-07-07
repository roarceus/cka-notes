# KubeConfig

## What It Does

kubeconfig stores cluster connection details, credentials, and context mappings so you don't need to pass `--server`, `--cert`, `--key` flags on every `kubectl` command.

Default location: `~/.kube/config`

---

## Structure — Three Sections

```yaml
apiVersion: v1
kind: Config
current-context: my-kube-admin@my-kube-playground   # active context

clusters:
- name: my-kube-playground
  cluster:
    server: https://my-kube-playground:6443
    certificate-authority: /etc/kubernetes/pki/ca.crt
    # or embed directly:
    # certificate-authority-data: <base64-encoded-ca.crt>

users:
- name: my-kube-admin
  user:
    client-certificate: admin.crt
    client-key: admin.key

contexts:
- name: my-kube-admin@my-kube-playground
  context:
    cluster: my-kube-playground
    user: my-kube-admin
    namespace: finance          # optional default namespace
```

| Section | Purpose |
|---|---|
| `clusters` | API server endpoints and CA certs |
| `users` | Credentials (certs/keys or tokens) |
| `contexts` | Links a cluster + user (+ optional namespace) |

---

## Essential Commands

```bash
# View current kubeconfig
kubectl config view

# View a specific kubeconfig file
kubectl config view --kubeconfig=my-custom-config

# Show current context
kubectl config current-context

# Switch context
kubectl config use-context prod-user@production

# List all contexts
kubectl config get-contexts

# Use a different kubeconfig file for one command
kubectl get pods --kubeconfig=my-custom-config

# Set kubeconfig via env var
export KUBECONFIG=/path/to/my-config
```

---

## Certificates in kubeconfig

### Path reference (recommended)
```yaml
certificate-authority: /etc/kubernetes/pki/ca.crt
client-certificate: /home/user/admin.crt
client-key: /home/user/admin.key
```

### Embedded base64 data
```yaml
certificate-authority-data: LS0tLS1CRUdJTi...   # base64 encoded
client-certificate-data: LS0tLS1CRUdJTi...
client-key-data: LS0tLS1CRUdJTi...
```

```bash
# Encode a cert for embedding
cat ca.crt | base64 -w 0

# Decode embedded cert data
echo "LS0tLS..." | base64 --decode
```

---

> [!tip] CKA Exam
> - Default kubeconfig is `~/.kube/config` — most exam tasks work here automatically
> - `kubectl config use-context <name>` to switch between clusters/users
> - `kubectl config get-contexts` to see all available contexts and which is active (`*`)
> - Add a default namespace to a context to avoid specifying `-n` on every command
> - Use `--kubeconfig` flag or `KUBECONFIG` env var to use an alternate config file
> - `certificate-authority-data` (with `-data`) = base64-encoded inline; `certificate-authority` (without `-data`) = file path — don't mix them up
