# Viewing Certificate Details

## Finding Where Certificates Are Configured

### kubeadm clusters (components run as static pods)
```bash
# View all cert flags for the API server
cat /etc/kubernetes/manifests/kube-apiserver.yaml

# All certs are under:
ls /etc/kubernetes/pki/
ls /etc/kubernetes/pki/etcd/
```

### Manual clusters (components run as systemd services)
```bash
cat /etc/systemd/system/kube-apiserver.service
```

---

## Inspecting a Certificate

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

Key things to check in the output:

| Field | What to look for |
|---|---|
| `Subject: CN=` | The certificate's name (e.g. `kube-apiserver`) |
| `Subject Alternative Names` | All DNS names and IPs the cert is valid for |
| `Issuer: CN=` | The CA that signed it (should be `kubernetes` on kubeadm) |
| `Not After` | Expiry date — confirm it hasn't expired |
| `Subject: O=` | Group membership (e.g. `system:masters` for admin) |

### Example — check API server cert
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A 5 "Subject\|Validity\|Alt"
```

---

## Certificate Health Check — What to Verify

For each certificate:
- ✅ Correct `CN` (common name matches expected component)
- ✅ Correct `O` (organization/group where applicable)
- ✅ Signed by the right CA
- ✅ Not expired (`Not After` in the future)
- ✅ Correct SANs (especially for kube-apiserver)

![Certificate inventory table](https://kodekloud.com/kk-media/image/upload/v1752869980/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-View-Certificate-Details/frame_210.jpg)

---

## Troubleshooting Certificate Issues — Logs

### kubeadm clusters (pods)
```bash
# Normal log access
kubectl logs kube-apiserver-controlplane -n kube-system

# If API server is down — use crictl or docker directly
crictl ps -a
crictl logs <container-id>

# Older clusters using docker
docker ps -a
docker logs <container-id>
```

### Manual clusters (systemd services)
```bash
journalctl -u kube-apiserver.service -l
journalctl -u etcd.service -l
```

Common TLS error in logs:
```
transport: authentication handshake failed: remote error: tls: bad certificate
```
→ Cert expired, wrong CA, or missing SAN.

---

> [!tip] CKA Exam
> - Start cert troubleshooting by finding cert paths from the component manifest/service file
> - `openssl x509 -in <cert> -text -noout` is the go-to command for inspecting any certificate
> - If `kubectl` is unresponsive (API server down), use `crictl logs` to debug
> - Common causes of TLS failures: expired cert, wrong CN/SAN, cert signed by wrong CA
> - On kubeadm clusters, all certs are in `/etc/kubernetes/pki/` — get familiar with this directory
