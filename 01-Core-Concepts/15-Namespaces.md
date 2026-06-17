# Namespaces

## What They Do

Namespaces **logically isolate resources** within a cluster. Useful for separating environments (dev/prod), teams, or applications on the same cluster.

---

## Built-in Namespaces

| Namespace | Purpose |
|---|---|
| `default` | Where resources go if no namespace is specified |
| `kube-system` | Core cluster components (DNS, API server, scheduler, etc.) |
| `kube-public` | Publicly accessible resources — rarely used directly |
| `kube-node-lease` | Holds node heartbeat lease objects — improves node failure detection |

---

## DNS Between Namespaces

Within the same namespace, reach a service by its short name:
```
db-service
```

Across namespaces, use the **fully qualified DNS name**:
```
db-service.dev.svc.cluster.local
```

Format: `<service-name>.<namespace>.svc.cluster.local`

---

## Essential Commands

```bash
# List resources in a specific namespace
kubectl get pods --namespace=kube-system
kubectl get pods -n dev                    # shorthand

# List across ALL namespaces
kubectl get pods --all-namespaces
kubectl get pods -A                        # shorthand

# Create a resource in a specific namespace
kubectl create -f pod-definition.yaml --namespace=dev

# Create a namespace
kubectl create namespace dev

# Switch default namespace for current context
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

---

## Namespace in YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev        # set here to always deploy to this namespace
  labels:
    app: myapp
spec:
  containers:
  - name: nginx-container
    image: nginx
```

---

## ResourceQuota — Limiting a Namespace

Prevents one namespace from consuming too many cluster resources:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

```bash
kubectl create -f compute-quota.yaml
```

---

> [!tip] CKA Exam
> - Always check which namespace you're in — many exam tasks fail because resources are in the wrong namespace
> - `-n <namespace>` is the shorthand for `--namespace=<namespace>`
> - `kubectl get pods -A` is the fastest way to see everything across all namespaces
> - Cross-namespace DNS format: `<service>.<namespace>.svc.cluster.local`
> - `kubectl config set-context --current --namespace=<ns>` switches your default namespace persistently for the session
