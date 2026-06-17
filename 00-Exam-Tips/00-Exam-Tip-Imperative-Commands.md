# Exam Tip — Imperative Commands & Dry Run

Avoid writing YAML from scratch in the exam. Use `--dry-run=client -o yaml` to generate a template, tweak it, then apply.

## Pods

```bash
# Create directly
kubectl run nginx --image=nginx

# Generate YAML without creating
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

## Deployments

```bash
# Create directly
kubectl create deployment nginx --image=nginx

# With replicas
kubectl create deployment nginx --image=nginx --replicas=4

# Scale existing deployment
kubectl scale deployment nginx --replicas=4

# Generate YAML without creating
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Save to file, edit, then apply
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
kubectl apply -f nginx-deployment.yaml
```

---

## Services

### ClusterIP

```bash
# Best option — auto-uses pod's labels as selectors
kubectl expose pod redis --port=6379 --name=redis-service --dry-run=client -o yaml

# Alternative — does NOT use pod labels (assumes app=redis); avoid unless labels match
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

### NodePort

```bash
# Best option — auto-uses pod's labels as selectors, BUT cannot set nodePort directly
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
# → save to file, manually add nodePort, then kubectl apply

# Alternative — can set nodePort, BUT does NOT use pod labels as selectors
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

> **Rule of thumb:** prefer `kubectl expose` (gets selectors right). If you need a specific `nodePort`, use `kubectl expose --dry-run=client -o yaml > svc.yaml`, add `nodePort` manually, then apply.

---

## The General Pattern

```bash
kubectl <command> <flags> --dry-run=client -o yaml > file.yaml
# edit file.yaml as needed
kubectl apply -f file.yaml
```

> [!tip] CKA Exam
> - `--dry-run=client` = validate locally, don't send to cluster
> - `-o yaml` = output the resource definition as YAML
> - `kubectl expose` auto-inherits pod labels as selectors — `kubectl create service` does not
> - Bookmarks for exam:
>   - https://kubernetes.io/docs/reference/kubectl/conventions/
>   - https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
