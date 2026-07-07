# Certificates API

## What It Does

The Certificates API automates CSR management — users submit signing requests via Kubernetes objects, admins approve them with `kubectl`, and Kubernetes signs the cert using the cluster CA. No manual file transfer needed.

Handled by the **Controller Manager** (specifically the CSR-Approving and CSR-Signing controllers), which has access to the CA key/cert:
```
--cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
--cluster-signing-key-file=/etc/kubernetes/pki/ca.key
```

---

## Full Workflow

### Step 1 — User generates a key and CSR
```bash
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

### Step 2 — Admin creates a CertificateSigningRequest object

Base64-encode the CSR first:
```bash
cat jane.csr | base64 -w 0    # -w 0 prevents line wrapping
```

Then create the object:
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  expirationSeconds: 86400    # 1 day
  usages:
  - digital signature
  - key encipherment
  - client auth             # use "server auth" for server certs
  request: <base64-encoded-CSR>
```

```bash
kubectl apply -f jane-csr.yaml
```

### Step 3 — View and approve

```bash
# List pending CSRs
kubectl get csr

# Approve
kubectl certificate approve jane

# Deny
kubectl certificate deny jane

# Delete
kubectl delete csr jane
```

### Step 4 — Extract the signed certificate

```bash
kubectl get csr jane -o jsonpath='{.status.certificate}' | base64 --decode > jane.crt
```

---

## CSR Object Quick Reference

```bash
# View all CSRs
kubectl get csr

# See full details including the encoded cert after approval
kubectl get csr jane -o yaml
```

---

> [!tip] CKA Exam
> - The CSR `request` field must be **base64-encoded** (use `cat jane.csr | base64 -w 0`)
> - `usages` must include `client auth` for user certs, `server auth` for server certs
> - `kubectl certificate approve <name>` is the key command — know it
> - After approval, extract the cert: `kubectl get csr <name> -o jsonpath='{.status.certificate}' | base64 --decode`
> - The Controller Manager signs CSRs using `--cluster-signing-cert-file` and `--cluster-signing-key-file`
> - `apiVersion: certificates.k8s.io/v1` — note the non-standard API group
