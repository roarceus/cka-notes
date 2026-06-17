# A Note on Docker Deprecation

## The Short Version

Docker is **deprecated as a Kubernetes runtime** (removed in v1.24), but **Docker itself is not dead**. It's still widely used for building images and local development.

---

## Why Kubernetes Dropped Docker

Docker was never just a runtime — it's a full platform bundling:
- CLI, API, build tools, volume support, security configs
- `runc` (the actual container runtime)
- `containerd` (the daemon managing runc)

Kubernetes only needs the runtime piece. Once `containerd` became CRI-compatible and available as a standalone component, there was no reason to carry the rest of Docker's overhead.

![Container runtimes and CRI](https://kodekloud.com/kk-media/image/upload/v1752869700/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-A-note-on-Docker-Deprecation/frame_70.jpg)

---

## What's Still True

- **Docker images** are OCI-compliant → work perfectly with `containerd` and Kubernetes
- **Docker** is still the most common tool for building images and local dev workflows
- **Kubernetes clusters** use `containerd` (or CRI-O) under the hood — not Docker

---

> [!tip] CKA Exam
> - "Docker deprecated" = deprecated **as a Kubernetes runtime**, not as a tool
> - Images built with Docker run fine on Kubernetes — no changes needed
> - On exam nodes, the runtime will be `containerd` — use `crictl` to inspect, not `docker`
