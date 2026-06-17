# Docker vs ContainerD

## Why This Matters

Kubernetes dropped Docker as a runtime in **v1.24**. Understanding the evolution explains why `containerd` is now the default runtime and which CLI tools to use when debugging nodes.

---

## The Evolution

- **Before CRI:** Kubernetes was tightly coupled to Docker. A workaround called **dockershim** was used to make Docker work with Kubernetes.
- **CRI (Container Runtime Interface):** Kubernetes introduced CRI to standardize how runtimes plug in — any runtime implementing CRI (and OCI standards) can work with Kubernetes.
- **ContainerD:** Originally part of Docker, now a standalone CNCF project. It is **natively CRI-compatible** — no shim needed.
- **Kubernetes v1.24:** Removed dockershim. Docker is no longer a supported runtime. Docker-built images still work fine (they're OCI-compliant).

![Docker, Kubernetes, and CRI relationship](https://kodekloud.com/kk-media/image/upload/v1752869705/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Docker-vs-ContainerD/frame_190.jpg)

---

## CLI Tools Comparison

![ctr vs nerdctl vs crictl](https://kodekloud.com/kk-media/image/upload/v1752869712/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Docker-vs-ContainerD/frame_750.jpg)

| Tool | Maintained By | Purpose | Use In Production? |
|---|---|---|---|
| `ctr` | ContainerD | Low-level debugging only | ❌ No |
| `nerdctl` | ContainerD | Docker-like general use | ✅ Yes |
| `crictl` | Kubernetes | Debug any CRI runtime | ⚠️ Debugging only |

### `ctr` — ContainerD's built-in CLI
Bundled with containerd. Limited features, not user-friendly. Only for quick debugging.
```bash
ctr images pull docker.io/library/redis:alpine
ctr run docker.io/library/redis:alpine redis
```

### `nerdctl` — Recommended for general use
Drop-in Docker CLI replacement for ContainerD. Same syntax, extra features (lazy pull, encrypted images, image signing).
```bash
# Just replace "docker" with "nerdctl"
nerdctl run --name redis redis:alpine
nerdctl run --name webserver -p 80:80 -d nginx
```

### `crictl` — Kubernetes community debug tool
Works with **any** CRI-compatible runtime. Used to inspect pods, containers, and logs on a node.
```bash
crictl pull busybox
crictl images
crictl ps -a
crictl logs <container-id>
crictl pods        # lists pods — docker CLI has no equivalent
```

> ⚠️ Containers created manually with `crictl` will be **deleted by kubelet** — they aren't part of any pod spec.

---

## Runtime Endpoints (Post v1.24)

The kubelet connects to the container runtime via a socket. Default endpoints:
```
unix:///run/containerd/containerd.sock   # containerd
unix:///run/crio/crio.sock               # CRI-O
unix:///var/run/cri-dockerd.sock         # Docker (via cri-dockerd adapter)
```

Set the endpoint manually when using `crictl`:
```bash
crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
# or
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/containerd/containerd.sock
```

---

> [!tip] CKA Exam
> - **`crictl` is the tool you'll use on exam nodes** — know its common commands (`ps`, `logs`, `pods`, `images`, `inspect`)
> - Remember: `crictl` is for **inspection/debugging**, not creating workloads
> - Docker was removed as a runtime in **v1.24** — but Docker images still work fine
> - If asked to debug a container runtime issue on a node, check the runtime endpoint and use `crictl`
