# Volumes

## Why Volumes Exist

Pods are ephemeral — when a pod is deleted, all data inside it is lost. Volumes let pods **persist data** beyond their own lifecycle by storing it outside the container.

---

## Volume in a Pod YAML

A volume is defined at the **pod level** and mounted into a container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt        # path inside the container
      name: data-volume
  volumes:
  - name: data-volume
    hostPath:                # volume type
      path: /data            # path on the host node
      type: Directory
```

---

## Volume Types

| Type | Description | Use case |
|---|---|---|
| `hostPath` | Uses a directory on the node | Single-node only — data tied to one node |
| `emptyDir` | Temporary dir, deleted with pod | Sharing data between containers in a pod |
| `configMap` | Mounts a ConfigMap as files | Config injection |
| `secret` | Mounts a Secret as files | Credential injection |
| `awsElasticBlockStore` | AWS EBS volume | Multi-node on AWS |
| `azureDisk` / `azureFile` | Azure storage | Multi-node on Azure |
| `gcePersistentDisk` | GCP Persistent Disk | Multi-node on GCP |
| `nfs` | NFS share | Shared storage across nodes |
| `persistentVolumeClaim` | References a PVC | Standard production approach |

---

## hostPath — Simple but Limited

```yaml
volumes:
- name: data-volume
  hostPath:
    path: /data
    type: Directory
```

> ⚠️ **Not suitable for multi-node clusters** — each node has its own `/data` directory. If a pod moves to a different node, it won't find the same data.

---

## emptyDir — Shared Within a Pod

Created when pod starts, deleted when pod terminates. Useful for sharing files between containers in the same pod:

```yaml
volumes:
- name: shared-data
  emptyDir: {}
```

---

> [!tip] CKA Exam
> - Volumes are defined at **pod level** under `volumes:`, then referenced in containers via `volumeMounts:`
> - `hostPath` is simple but node-specific — for real multi-node storage use PVCs (next note)
> - `emptyDir` is the go-to for sharing temp data between init containers and main containers
> - The recommended production pattern: `persistentVolumeClaim` → references a PV → abstracts the storage backend
