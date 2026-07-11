# Developing Network Policies

Practical patterns for writing NetworkPolicy YAML. See [[59-Network-Policies]] for the conceptual overview.

---

## Pattern 1 — Deny All Ingress (no rules = block all)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  # no ingress rules listed = deny ALL ingress
```

---

## Pattern 2 — Allow Ingress from Specific Pod + Namespace (AND) + External IP (OR)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:          # ← same dash item as namespaceSelector
        matchLabels:
          name: api-pod
      namespaceSelector:    # AND — pod must be in prod namespace too
        matchLabels:
          name: prod
    - ipBlock:              # ← separate dash item = OR
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
```

> Traffic is allowed if it comes from (api-pod **AND** in prod namespace) **OR** from 192.168.5.10

---

## Pattern 3 — Ingress + Egress (full bidirectional control)

Example: DB pod accepts from API, sends backups to external server.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          name: prod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```

---

## AND vs OR — Quick Reference

| Structure | Logic |
|---|---|
| Two selectors on the **same** `-` item | **AND** — both must match |
| Two items under **separate** `-` dashes | **OR** — either can match |

```yaml
from:
- podSelector: ...   # AND
  namespaceSelector: ...
- ipBlock: ...       # OR (separate -)
```

---

> [!tip] CKA Exam
> - No ingress rules listed = **deny all ingress** (not allow all) — when `Ingress` is in `policyTypes`
> - `ports` at the ingress/egress rule level applies to **all** `from`/`to` selectors in that rule
> - Always label your namespaces if you want to use `namespaceSelector` — unlabelled namespaces can't be selected
> - To allow DNS (needed if pods do hostname lookups): add egress rule for port 53 UDP/TCP to `kube-dns`
