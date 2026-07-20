# Prerequisite — DNS

Essential DNS concepts needed for understanding Kubernetes service discovery and CoreDNS.

---

## Name Resolution Order

Linux resolves hostnames in this order (configured in `/etc/nsswitch.conf`):

```bash
cat /etc/nsswitch.conf
hosts: files dns    # check /etc/hosts first, then DNS server
```

---

## Key Config Files

### `/etc/hosts` — local overrides
```
192.168.1.11    db
192.168.1.11    db.mycompany.com
```

### `/etc/resolv.conf` — DNS server config
```
nameserver 192.168.1.100    # DNS server IP
search mycompany.com        # appended to short names: "web" → "web.mycompany.com"
```

---

## DNS Record Types

| Type | Purpose | Example |
|---|---|---|
| `A` | Hostname → IPv4 | `web-server → 192.168.1.1` |
| `AAAA` | Hostname → IPv6 | `web-server → 2001:db8::1` |
| `CNAME` | Alias → another hostname | `food.web → eat.web` |

---

## DNS Lookup Tools

```bash
# nslookup — queries DNS only (ignores /etc/hosts)
nslookup www.google.com

# dig — detailed DNS query info
dig www.google.com

# Check resolution including /etc/hosts
ping db
```

> `nslookup` and `dig` **bypass** `/etc/hosts` — they only talk to the DNS server.

---

## Why This Matters for Kubernetes

- Kubernetes uses **CoreDNS** as its internal DNS server
- Services are reachable by name (e.g. `db-service.default.svc.cluster.local`)
- The search domain in pods' `/etc/resolv.conf` is set to `cluster.local` — so short names like `db-service` resolve automatically
- DNS troubleshooting: `nslookup <service>` or `dig <service>` from inside a pod

---

> [!tip] CKA Exam
> - `/etc/resolv.conf` on pods is auto-configured by Kubernetes — contains `nameserver` (CoreDNS IP) and `search` domains
> - Use `nslookup <service-name>` inside a pod to test DNS resolution
> - `nslookup` ignores `/etc/hosts` — useful for isolating DNS vs local resolution issues
> - CoreDNS runs as a Deployment in `kube-system` — check it with `kubectl get pods -n kube-system`
