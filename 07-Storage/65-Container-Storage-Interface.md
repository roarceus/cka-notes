# Container Storage Interface (CSI)

## What It Is

CSI is a **standard interface** that allows any container orchestrator to work with any storage system — without embedding storage-specific code in the orchestrator itself.

The same idea as CRI (container runtimes) and CNI (networking) — but for **storage**.

| Interface | Standardises |
|---|---|
| CRI | Container runtimes (containerd, CRI-O) |
| CNI | Network plugins (Calico, Flannel, Weave) |
| **CSI** | **Storage drivers** (EBS, Azure Disk, Portworx…) |

---

## How It Works

When Kubernetes needs to provision storage for a pod, it calls CSI RPCs (Remote Procedure Calls) defined by the standard:

- **CreateVolume** — provision a new volume on the storage backend
- **DeleteVolume** — remove a volume from the storage backend
- **NodePublishVolume** / **NodeUnpublishVolume** — mount/unmount on the node

The storage driver implements these RPCs — Kubernetes doesn't care which backend is used.

![CSI RPC diagram](https://kodekloud.com/kk-media/image/upload/v1752869982/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Container-Storage-Interface/frame_150.jpg)

---

## CSI is Not Kubernetes-Specific

Any orchestrator implementing CSI works with any CSI-compliant storage driver. Kubernetes, Cloud Foundry, and Mesos all support CSI.

Popular CSI drivers: AWS EBS, Azure Disk, GCP Persistent Disk, Portworx, NetApp, Pure Storage, Dell EMC.

---

> [!tip] CKA Exam
> - CSI = standard for plugging storage backends into Kubernetes — same pattern as CRI and CNI
> - You don't configure CSI directly — it works behind the scenes when you use StorageClasses
> - If a storage provisioner is available in your cluster, it's a CSI driver doing the work
> - Know the parallel: CRI (runtimes) → CNI (networking) → CSI (storage)
