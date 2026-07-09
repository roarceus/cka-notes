# Role Based Access Controls (RBAC)

## Scope

**Role** + **RoleBinding** = namespace-scoped
**ClusterRole** + **ClusterRoleBinding** = cluster-wide (covered in next note)

---

## Role YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default      # omit for current namespace
rules:
- apiGroups: [""]         # "" = core API group (pods, services, configmaps, etc.)
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
- apiGroups: ["apps"]     # named group for deployments
  resources: ["deployments"]
  verbs: ["get", "list"]
  resourceNames: ["blue", "orange"]   # optional: restrict to specific resource names
```

---

## RoleBinding YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
  namespace: default
subjects:
- kind: User               # User | Group | ServiceAccount
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role               # Role | ClusterRole
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

---

## Imperative Commands

```bash
# Create a role
kubectl create role developer \
  --verb=list,get,create,update,delete \
  --resource=pods

# Create a rolebinding
kubectl create rolebinding devuser-developer-binding \
  --role=developer \
  --user=dev-user
```

---

## Viewing & Verifying

```bash
# List roles and bindings
kubectl get roles
kubectl get rolebindings

# Describe
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding

# Test your own permissions
kubectl auth can-i create deployments
kubectl auth can-i delete nodes

# Test another user's permissions (as admin)
kubectl auth can-i create pods --as dev-user
kubectl auth can-i create pods --as dev-user --namespace test
```

---

## Key Rules

- `apiGroups: [""]` → core group (pods, services, nodes, configmaps, secrets, etc.)
- `apiGroups: ["apps"]` → deployments, replicasets, statefulsets, daemonsets
- `resourceNames: ["blue", "orange"]` → restrict to specific named resources only
- A role can have **multiple rules** — each rule is a separate entry in the list

---

> [!tip] CKA Exam
> - `kubectl auth can-i <verb> <resource> --as <user>` is the fastest way to verify permissions
> - Always specify `namespace` in Role/RoleBinding metadata if not working in `default`
> - Imperative: `kubectl create role` + `kubectl create rolebinding` is faster than writing YAML
> - Core API group resources use `apiGroups: [""]` — this empty string is easy to forget
> - `resourceNames` restricts to specific instances — not just the resource type
> - If a user gets `Forbidden` errors, check: does the Role exist? Does the RoleBinding reference the correct user/role? Is it in the right namespace?
