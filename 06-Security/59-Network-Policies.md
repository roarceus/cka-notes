# Network Policies

## Default Behaviour

By default, Kubernetes uses an **"all-allow"** rule — every pod can communicate with every other pod in the cluster. Network Policies let you restrict this.

---

## Ingress vs Egress

| Type | Direction | Example |
|---|---|---|
| **Ingress** | Traffic **coming into** the pod | User → Web server on port 80 |
| **Egress** | Traffic **going out from** the pod | API server → DB on port 3306 |

> Response traffic (dotted arrows) is automatically allowed — you only define rules for the originating direction.

---

## How Network Policies Work

- Attach a policy to pods using **labels and selectors**
- Once a NetworkPolicy selects a pod, **only traffic explicitly allowed by the policy is permitted** (for the specified `policyTypes`)
- Unspecified `policyTypes` remain fully open (e.g. if you only specify `Ingress`, egress is still all-allow)

---

## NetworkPolicy YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db          # applies to pods with this label
  policyTypes:
  - Ingress             # only restrict ingress; egress remains open
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod  # only allow traffic from api-pod
      namespaceSelector:  # optional: restrict to a specific namespace
        matchLabels:
          name: prod
    - ipBlock:            # or allow a specific IP range
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

### Egress example

```yaml
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: db
    ports:
    - protocol: TCP
      port: 3306
```

---

## Selector Combinations

Multiple entries under `from` (or `to`) are **OR** conditions — traffic is allowed if it matches **any** entry:

```yaml
  ingress:
  - from:
    - podSelector:        # source 1
        matchLabels:
          name: api-pod
    - ipBlock:            # source 2 — OR condition
        cidr: 10.0.0.0/8
```

Multiple fields **within one entry** are **AND** conditions:

```yaml
  - from:
    - podSelector:        # AND
        matchLabels:
          name: api-pod
      namespaceSelector:  # both must match (same dash level = AND)
        matchLabels:
          name: prod
```

---

## Commands

```bash
kubectl create -f network-policy.yaml
kubectl get networkpolicies
kubectl get netpol             # shorthand
kubectl describe networkpolicy db-policy
```

---

> [!tip] CKA Exam
> - Network Policies are enforced by the **CNI plugin** — Flannel does **not** support them; Calico, Weave, Cilium do
> - `podSelector: {}` (empty) = select **all pods** in the namespace
> - If only `Ingress` is in `policyTypes`, egress traffic is unrestricted (and vice versa)
> - `namespaceSelector` restricts the **source namespace** — without it, matching pods from any namespace are allowed
> - Multiple `from` list items = **OR**; multiple fields within one item = **AND** — this distinction is a common exam trap
> - `ipBlock` allows/denies specific CIDR ranges — useful for allowing external traffic
