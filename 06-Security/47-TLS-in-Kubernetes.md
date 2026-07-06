# TLS in Kubernetes

## The Big Picture

Every component in a Kubernetes cluster communicates over TLS. Each component needs:
- A **server certificate** (if it exposes an endpoint)
- A **client certificate** (if it calls another component's endpoint)
- All certificates must be signed by a trusted **CA**

---

## Certificate Map

### Server Certificates (components exposing HTTPS endpoints)

| Component | Certificate | Key |
|---|---|---|
| kube-apiserver | `apiserver.crt` | `apiserver.key` |
| etcd | `etcd-server.crt` | `etcd-server.key` |
| kubelet (each node) | `kubelet.crt` | `kubelet.key` |

### Client Certificates (components calling the API server)

| Client | Certificate | Key |
|---|---|---|
| admin (kubectl) | `admin.crt` | `admin.key` |
| kube-scheduler | `scheduler.crt` | `scheduler.key` |
| kube-controller-manager | `controller-manager.crt` | `controller-manager.key` |
| kube-proxy | `kube-proxy.crt` | `kube-proxy.key` |
| kube-apiserver → etcd | `apiserver-etcd-client.crt` | `apiserver-etcd-client.key` |
| kube-apiserver → kubelet | `apiserver-kubelet-client.crt` | `apiserver-kubelet-client.key` |

### Certificate Authority

| | Certificate | Key |
|---|---|---|
| Cluster CA | `ca.crt` | `ca.key` |

> A cluster needs at least one CA. ETCD may use its own dedicated CA for additional isolation.

---

## Visual Overview

![All Kubernetes component certificates](https://kodekloud.com/kk-media/image/upload/v1752869979/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-TLS-in-Kubernetes/frame_420.jpg)

---

## Key Observations

- The **kube-apiserver** is both a server (serving kubectl) and a client (calling etcd and kubelets) — so it has multiple certificate pairs
- **kubelet** is both a server (the API server calls it) and a client (it calls the API server)
- All client certs are signed by the **same cluster CA** so the API server trusts them
- The `ca.crt` (public) is distributed to all components so they can verify each other's certificates

---

> [!tip] CKA Exam
> - Know **which components need which certs** — especially kube-apiserver's multiple pairs
> - On kubeadm clusters, all certs live under `/etc/kubernetes/pki/`
> - etcd certs are under `/etc/kubernetes/pki/etcd/`
> - The CA cert (`ca.crt`) is the root of trust — if you're troubleshooting TLS errors, check that each component is using certs signed by the same CA
> - Cert naming reminder: `*.crt` / `*.pem` = certificate (public); `*.key` / `*-key.pem` = private key
