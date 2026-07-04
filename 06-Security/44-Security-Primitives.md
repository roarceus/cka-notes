# Kubernetes Security Primitives

## The Two Core Questions

Every security decision in Kubernetes comes down to:
1. **Who can access the cluster?** → Authentication
2. **What can they do?** → Authorization

---

## Layers of Security

### Host Security (foundation)
- Disable root access and password-based SSH
- Use SSH key-based authentication only
- A compromised host = compromised cluster

### API Server Access Control

The kube-apiserver is the single gateway to everything. All kubectl commands, internal component communication, and external access flows through it.

**Authentication — who can access?**

| Mechanism | Used for |
|---|---|
| Static username/password or token files | Simple/legacy (not recommended) |
| Client certificates (x509) | kubectl users, kubeconfig |
| External providers (LDAP, OIDC) | Enterprise integration |
| Service Accounts | Pods and in-cluster processes |

**Authorization — what can they do?**

| Mechanism | Description |
|---|---|
| **RBAC** | Role-based — standard and recommended |
| ABAC | Attribute-based — rarely used |
| Node Authorization | Used by kubelets specifically |
| Webhook | External authorization service |

### TLS Encryption (component communication)
All communication between cluster components is encrypted with TLS:
- etcd ↔ API server
- API server ↔ kubelet, scheduler, controller manager
- kubectl ↔ API server

![TLS between Kubernetes components](https://kodekloud.com/kk-media/image/upload/v1752869940/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Kubernetes-Security-Primitives/frame_160.jpg)

### Network Policies
By default, all pods can talk to all other pods. Network Policies restrict traffic at the network level — covered in depth in the Networking section.

---

> [!tip] CKA Exam
> - Security in Kubernetes is built in layers: host → API server auth → API server authz → TLS → network policies
> - **RBAC** is the standard authorization mechanism — know it well (covered in upcoming notes)
> - Service Accounts are the authentication mechanism **for pods** — not for human users
> - TLS certificates are used everywhere between components — cert paths matter for troubleshooting
