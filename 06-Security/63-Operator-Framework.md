# Operator Framework

## What It Is

An **Operator** packages a CRD + custom controller into a single deployable unit. Instead of creating them separately, one `kubectl create -f operator.yaml` sets everything up automatically.

```
Operator deployment
    → creates the CRD
    → deploys the custom controller
    → manages the full application lifecycle
```

---

## Why Operators Exist

Operators encode **operational knowledge** into software — they handle tasks that would otherwise require a human admin:

- Application installation
- Configuration management
- Upgrades and rollbacks
- Backups and restores
- Disaster recovery
- Troubleshooting

---

## Real-World Example — etcd Operator

The etcd operator manages an etcd cluster using three CRDs:

| CRD | Purpose |
|---|---|
| `EtcdCluster` | Deploy and manage an etcd cluster |
| `EtcdBackup` | Trigger and manage backups |
| `EtcdRestore` | Restore from a backup |

Each CRD has its own controller handling the reconciliation logic.

---

## Deploying with an Operator

```bash
# 1. Install the Operator Lifecycle Manager (OLM)
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.19.1/install.sh | bash -s v0.19.1

# 2. Deploy an operator (e.g. etcd)
kubectl create -f https://operatorhub.io/install/etcd.yaml

# 3. Check installed operators
kubectl get csv -n my-etcd    # csv = ClusterServiceVersion
```

Browse available operators at: **https://operatorhub.io/**
Popular apps with operators: etcd, MySQL, Prometheus, Grafana, Argo CD, Istio.

---

> [!tip] CKA Exam
> - Operators = CRD + Controller bundled together — conceptual understanding is the exam focus
> - You won't build an operator in the exam — but knowing what they are and how they work is expected
> - The exam primarily tests **CRDs** directly — operators are supplemental context
> - `kubectl get csv` lists installed ClusterServiceVersions (operator installations)
