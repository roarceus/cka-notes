# TLS Certificate Creation in Kubernetes

## The General Process (OpenSSL)

Every certificate follows the same three steps:

```bash
# 1. Generate a private key
openssl genrsa -out <name>.key 2048

# 2. Generate a Certificate Signing Request (CSR)
openssl req -new -key <name>.key -subj "/CN=<name>" -out <name>.csr

# 3. Sign with the CA to produce the certificate
openssl x509 -req -in <name>.csr -CA ca.crt -CAkey ca.key -out <name>.crt
```

---

## Step 1 — Create the CA (self-signed)

```bash
openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt   # self-signed
```

The CA's `ca.key` and `ca.crt` are used to sign all other certificates.

---

## Step 2 — Create Client Certificates

### Admin user
```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key \
  -subj "/CN=kube-admin/O=system:masters" \   # O= sets the group
  -out admin.csr
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

> `O=system:masters` grants cluster-admin rights — the group membership is embedded in the certificate.

Same process for: `scheduler`, `controller-manager`, `kube-proxy` — prefix CN with `system:` (e.g. `CN=system:kube-scheduler`).

### Using a client cert with curl
```bash
curl https://kube-apiserver:6443/api/v1/pods \
  --key admin.key --cert admin.crt --cacert ca.crt
```

---

## Step 3 — Create Server Certificates

### etcd server
Referenced in etcd config/manifest:
```
--cert-file=/path/to/etcdserver.crt
--key-file=/path/to/etcdserver.key
--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
--peer-cert-file=/path/to/etcdpeer1.crt      # for HA etcd clusters
--peer-key-file=/etc/kubernetes/pki/etcd/peer.key
```

### kube-apiserver (needs SANs — multiple names/IPs)

The API server is reachable by many names, so its cert needs **Subject Alternative Names**:

```bash
# First create an openssl config file with SANs
cat > openssl.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[v3_req]
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1          # ClusterIP of kubernetes service
IP.2 = 172.17.0.87        # API server node IP
EOF

openssl genrsa -out apiserver.key 2048
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" \
  -config openssl.cnf -out apiserver.csr
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key \
  -extensions v3_req -extfile openssl.cnf -out apiserver.crt
```

### kubelet (one per node)
Each node gets its own cert, named after the node. As a client of the API server, the CN must follow the format `system:node:<nodename>` and group `O=system:nodes`:

```bash
openssl req -new -key node01.key \
  -subj "/CN=system:node:node01/O=system:nodes" \
  -out node01.csr
```

---

> [!tip] CKA Exam
> - Know the **3-step process**: genrsa → req (CSR) → x509 (sign)
> - Group membership is set with `O=` in the CSR subject — `system:masters` = cluster admin
> - kube-apiserver cert **must have SANs** for all its DNS names and IPs — missing SANs = TLS errors
> - System component CNs are prefixed with `system:` (e.g. `system:kube-scheduler`)
> - On kubeadm clusters, you rarely generate certs manually — `kubeadm` handles it; the exam tests understanding more than manual generation
> - All certs on kubeadm clusters are in `/etc/kubernetes/pki/`
