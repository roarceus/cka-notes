# Environment Variables

## Setting Env Vars in a Pod

Three ways to set environment variables in a container:

### 1. Direct value (plain key-value)
```yaml
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    env:
    - name: APP_COLOR
      value: pink
    - name: APP_MODE
      value: prod
```

### 2. From a ConfigMap
```yaml
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: my-configmap
          key: APP_COLOR
```

### 3. From a Secret
```yaml
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: DB_PASSWORD
```

> `env` is an **array** — each item starts with `-` and has a `name` field. Use `value` for direct, `valueFrom` for ConfigMap/Secret references.

---

> [!tip] CKA Exam
> - `env` goes under the container spec, not at pod level
> - Direct `value` is fine for non-sensitive config; use `secretKeyRef` for passwords/tokens
> - ConfigMaps and Secrets are covered in the next notes — `valueFrom` is how you inject them into containers
