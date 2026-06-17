# ETCD for Beginners

## What is etcd?

A **distributed, reliable key-value store** — fast and simple. In Kubernetes, it's the single source of truth for all cluster state.

---

## Key-Value Store vs Relational DB

| Relational DB | Key-Value Store |
|---|---|
| Data in tables (rows & columns) | Data as independent documents |
| Schema must be predefined | Flexible, dynamic structure per entry |
| Adding new fields affects all rows | Each document can have different fields |

Data is stored as JSON/YAML documents, e.g.:
```json
{
  "name": "John Doe",
  "age": 45,
  "salary": 5000
}
```

---

## etcdctl — The CLI Client

etcd listens on port **2379** by default. You interact with it via `etcdctl`.

### API v2 vs v3 — Critical Difference

etcdctl defaults to **API v2** on older installs. Always verify and set to v3:

```bash
# Check current version and API
./etcdctl --version
# etcdctl version: 3.3.11
# API version: 2   ← bad, switch to v3

# Option 1: set for one command
ETCDCTL_API=3 ./etcdctl version

# Option 2: set for the session (preferred)
export ETCDCTL_API=3
```

### Core Commands

| Operation | API v2 | API v3 |
|---|---|---|
| Store a value | `etcdctl set key1 value1` | `etcdctl put key1 value1` |
| Retrieve a value | `etcdctl get key1` | `etcdctl get key1` |

```bash
# v3 examples
export ETCDCTL_API=3

etcdctl put key1 value1    # store
etcdctl get key1           # retrieve → prints key then value
```

---

> [!tip] CKA Exam
> - Always use **API v3** — set `export ETCDCTL_API=3` before running any `etcdctl` commands
> - The key command change from v2 → v3: `set` becomes **`put`**; `get` stays the same
> - etcd runs on port **2379**
> - In the exam you'll often need to pass extra flags to authenticate with etcd (certs) — covered in the next topic
