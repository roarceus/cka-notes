# ConfigMaps

## What They Do

ConfigMaps store **non-sensitive configuration data** as key-value pairs, decoupled from pod definitions. Inject them into pods as environment variables or mounted files.

---

## Creating ConfigMaps

### Imperative

```bash
# From literal values
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod

# From a file
kubectl create configmap app-config --from-file=app-config.properties
```

### Declarative (YAML)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```bash
kubectl apply -f configmap.yaml
```

---

## Viewing ConfigMaps

```bash
kubectl get configmaps
kubectl get cm               # shorthand

kubectl describe configmap app-config   # shows data section
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
    - configMapRef:
        name: app-config     # injects ALL key-value pairs as env vars
```

### Method 2 — Single specific key
```yaml
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

### Method 3 — Mount as a volume (files)
```yaml
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config       # each key becomes a file in /etc/config/
```

---

> [!tip] CKA Exam
> - Use `envFrom` to inject all keys at once — cleaner than listing each one
> - ConfigMaps are **not encrypted** — never store passwords/tokens in them (use Secrets)
> - `kubectl create configmap` with `--from-literal` is the fastest imperative approach
> - Each key in a volume-mounted ConfigMap becomes a **separate file** in the mount path
