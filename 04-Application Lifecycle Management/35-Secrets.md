# Secrets

## What They Are

Secrets store **sensitive data** (passwords, tokens, keys) separately from pod definitions. Similar to ConfigMaps but:
- Values are **Base64-encoded** (not encrypted by default)
- Access can be restricted via RBAC
- Kubernetes treats them with slightly more care (e.g. not shown in `describe`)

> ⚠️ Base64 is **encoding, not encryption** — anyone with cluster access can decode it. Enable etcd encryption at rest for real security.

---

## Creating Secrets

### Imperative
```bash
# From literal values (Kubernetes encodes automatically)
kubectl create secret generic app-secret \
  --from-literal=DB_Host=mysql \
  --from-literal=DB_User=root \
  --from-literal=DB_Password=paswrd

# From a file
kubectl create secret generic app-secret --from-file=app_secret.properties
```

### Declarative (YAML — must Base64-encode values yourself)
```bash
# Encode values first
echo -n 'mysql' | base64    # bXlzcWw=
echo -n 'root' | base64     # cm9vdA==
echo -n 'paswrd' | base64   # cGFzd3Jk
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw=
  DB_User: cm9vdA==
  DB_Password: cGFzd3Jk
```

```bash
kubectl apply -f secret.yaml
```

---

## Viewing Secrets

```bash
kubectl get secrets

# Describe — hides values
kubectl describe secret app-secret

# Show encoded values
kubectl get secret app-secret -o yaml

# Decode a value
echo -n 'cGFzd3Jk' | base64 --decode
```

---

## Injecting into a Pod

### Method 1 — All keys as env vars (envFrom)
```yaml
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    envFrom:
    - secretRef:
        name: app-secret       # injects all keys as env vars
```

### Method 2 — Single specific key
```yaml
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_Password
```

### Method 3 — Mount as volume (each key = a file)
```yaml
    volumeMounts:
    - name: secret-vol
      mountPath: /opt/app-secret-volumes
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: app-secret
```

```bash
# Each key appears as a file
ls /opt/app-secret-volumes
# DB_Host  DB_User  DB_Password

cat /opt/app-secret-volumes/DB_Password
# paswrd
```

---

> [!tip] CKA Exam
> - Use `kubectl create secret generic` — the imperative command handles Base64 encoding for you
> - YAML approach requires **manually Base64-encoding** values first with `echo -n 'value' | base64`
> - `kubectl get secret -o yaml` to see encoded values; pipe through `base64 --decode` to read them
> - Secrets are **Opaque** type by default — other types exist (e.g. `kubernetes.io/tls`, `kubernetes.io/dockerconfigjson`)
> - Never store secret YAML files in public git repos — the Base64 is trivially reversible
