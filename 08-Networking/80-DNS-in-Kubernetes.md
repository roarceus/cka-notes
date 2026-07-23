# DNS in Kubernetes

## Overview

- Kubernetes deploys a built-in DNS server by default, providing name resolution for **services** and (optionally) **pods** across the cluster
- Even though pods can always be reached directly by IP, DNS names give stable, human-readable ways to reach them — especially useful since pod IPs change on recreation

![Kube DNS resolving hostnames/IPs across pods and services](https://kodekloud.com/kk-media/image/upload/v1752869846/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-DNS-in-kubernetes/frame_160.jpg)

---

## Service DNS Records

- Every service automatically gets a DNS record mapping its name to its cluster IP
- Pods in the **same namespace** as the service can reach it with just the short name:

```bash
curl http://web-service
```

- Pods in a **different namespace** must qualify the name with the service's namespace:

```bash
curl http://web-service.apps
```

- The full FQDN adds the `svc` subdomain and the cluster's root domain (`cluster.local` by default):

```bash
curl http://web-service.apps.svc.cluster.local
```

- DNS naming structure: `<service>.<namespace>.svc.<cluster-domain>`
  - Each namespace gets its own subdomain
  - All services in a namespace are grouped under `svc`
  - The whole cluster shares a root domain (default `cluster.local`)

---

## Pod DNS Records

- Pod DNS records are **not created by default** — must be explicitly enabled
- When enabled, a pod's IP is converted into a hostname by replacing dots (`.`) with dashes (`-`), then combined with namespace, `pod` type, and the root domain:
  - Pod IP `10.244.2.5` in namespace `apps` → `10-244-2-5.apps.pod.cluster.local`

```bash
curl http://10-244-2-5.apps.pod.cluster.local
```

---

> [!tip] CKA Exam
> - Full service FQDN format: **`<service>.<namespace>.svc.cluster.local`**
> - Same-namespace pods can use just the **short service name** — no namespace or FQDN needed
> - Cross-namespace access requires at least `<service>.<namespace>`
> - Pod DNS records are **disabled by default** and use dashes instead of dots in the IP: `10-244-2-5.<namespace>.pod.cluster.local`
> - Default cluster root domain is **`cluster.local`**
