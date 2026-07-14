# Storage in Docker

## Docker's Layered Image Architecture

Each Dockerfile instruction creates a new **read-only layer** — only the diff from the previous layer is stored. Identical layers are **cached and shared** across images, saving disk space and build time.

```
Layer 5: ENTRYPOINT          ← unique to this image
Layer 4: COPY app.py         ← unique to this image
Layer 3: pip install flask   ← reused if same in another image
Layer 2: apt-get install     ← reused
Layer 1: ubuntu base image   ← reused
```

## Container Writable Layer (Copy-on-Write)

When you run a container, Docker adds a thin **writable layer** on top of the read-only image layers:

- Modifications to files are **copied to the writable layer first**, then modified there
- Original image layers stay untouched
- When the container is deleted, the writable layer and all its data are **lost**

![Copy-on-Write diagram](https://kodekloud.com/kk-media/image/upload/v1752869991/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Storage-in-Docker/frame_410.jpg)

---

## Persistent Data

| Type | Command | Data location |
|---|---|---|
| **Volume mount** | `-v data_volume:/var/lib/mysql` | `/var/lib/docker/volumes/` |
| **Bind mount** | `-v /data/mysql:/var/lib/mysql` | Any host directory |

```bash
# Create and use a named volume
docker volume create data_volume
docker run -v data_volume:/var/lib/mysql mysql

# Bind mount (use existing host directory)
docker run -v /data/mysql:/var/lib/mysql mysql

# Explicit syntax (preferred)
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```

---

## Storage Drivers

Manage the layered filesystem and copy-on-write mechanism. Docker selects the best driver automatically based on OS:

| Driver | Common on |
|---|---|
| `overlay2` | Ubuntu, most modern Linux (default) |
| `aufs` | Older Ubuntu |
| `devicemapper` | CentOS/Fedora |
| `btrfs`, `zfs` | Specialised setups |

---

## Storage Drivers vs Volume Driver Plugins

| | Storage Drivers | Volume Driver Plugins |
|---|---|---|
| Manages | Image layers + container writable layer | Persistent volumes |
| Examples | `overlay2`, `aufs`, `devicemapper` | `local`, `rexray/ebs`, `convoy`, `portworx` |
| Default | `overlay2` | `local` (stores in `/var/lib/docker/volumes/`) |

Volume driver plugins allow volumes to be backed by **third-party storage** (AWS EBS, GCP Persistent Disk, NetApp, Azure File Storage, etc.):

```bash
# Use Rex Ray EBS driver to provision volume from AWS EBS
docker run -it \
  --name mysql \
  --volume-driver rexray/ebs \
  --mount src=ebs-vol,target=/var/lib/mysql \
  mysql
```

---

> [!tip] CKA Exam
> - Container writable layer is **ephemeral** — data is lost when container is removed
> - Volumes persist beyond the container lifecycle — use them for databases and stateful data
> - This Docker knowledge is prerequisite for Kubernetes Volumes, PVs, and PVCs (covered in next notes)
> - Docker stores everything under `/var/lib/docker/` on the host
