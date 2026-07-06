# TLS Basics

## Symmetric vs Asymmetric Encryption

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | Same key encrypts and decrypts | Public key + Private key pair |
| Problem | Must share the key securely | Private key never leaves the owner |
| Used for | Bulk data encryption (fast) | Securely exchanging the symmetric key |

**In practice:** TLS uses asymmetric encryption to securely exchange a symmetric key, then uses that symmetric key for the actual session (best of both).

---

## How HTTPS Works (simplified)

1. Server sends its **public key** embedded in a certificate
2. Browser generates a **symmetric session key**, encrypts it with the server's public key
3. Server decrypts the session key using its **private key**
4. All further communication uses the symmetric session key

---

## Certificate Authorities (CAs)

CAs sign certificates to prove a server is legitimate. Browsers trust a pre-installed list of public CAs (Symantec, DigiCert, GlobalSign, etc.).

### Generating a Certificate Signing Request (CSR)

```bash
# 1. Generate a private key
openssl genrsa -out my-bank.key 2048

# 2. Generate a CSR
openssl req -new -key my-bank.key -out my-bank.csr \
  -subj "/C=US/ST=CA/O=MyOrg/CN=my-bank.com"

# 3. Send the CSR to a CA → they sign it and return a .crt
```

For internal use (e.g. Kubernetes), you act as your own **private CA**.

---

## File Naming Conventions — Important!

| File type | Typical extensions | Contains |
|---|---|---|
| Certificate (public key) | `.crt`, `.pem` | Public key + identity info |
| Private key | `.key`, `-key.pem` | Private key — never share |

Examples:
```
server.crt / server.pem       ← certificate (public)
server.key / server-key.pem   ← private key
ca.crt                        ← CA certificate
```

> In Kubernetes, if a filename contains `key` → it's a private key. If it ends in `.crt` or `.pem` without `key` → it's a certificate.

---

![Certificate and key file types](https://kodekloud.com/kk-media/image/upload/v1752869970/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-TLS-Basics/frame_1160.jpg)

---

> [!tip] CKA Exam
> - Know the file extension convention: `.crt`/`.pem` = certificate (public); `.key`/`-key.pem` = private key
> - Kubernetes uses a **private CA** to sign all internal component certificates
> - The CSR process: generate key → generate CSR → CA signs it → you get a certificate
> - Private keys are **never shared** — only the certificate (containing the public key) is distributed
