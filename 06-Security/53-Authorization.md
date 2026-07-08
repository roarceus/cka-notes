# Authorization

## Authorization Mechanisms

| Mechanism | Description |
|---|---|
| **Node** | Authorizes kubelet requests — kubelets with `system:node:*` certs in `system:nodes` group |
| **RBAC** | Role-based — standard production approach; roles define permissions, users/groups are bound to roles |
| **ABAC** | Attribute-based — JSON policy files per user; requires API server restart to update; legacy |
| **Webhook** | External tool (e.g. Open Policy Agent) decides access; API server sends request details, receives allow/deny |
| **AlwaysAllow** | Permits everything — default if no mode specified |
| **AlwaysDeny** | Denies everything |

---

## Configuring Authorization Mode

Set on the kube-apiserver via `--authorization-mode`:

```bash
# Single mode
--authorization-mode=RBAC

# Multiple modes — evaluated in order, first approval wins
--authorization-mode=Node,RBAC,Webhook
```

### How multiple modes work

Requests are checked against each authorizer **in order**. As soon as one **approves**, access is granted and remaining authorizers are skipped. If one **denies**, it passes to the next.

Standard production setting: `--authorization-mode=Node,RBAC`

---

## Checking the Current Authorization Mode

```bash
# On kubeadm clusters
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep authorization-mode

# Or check the running process
ps -aux | grep kube-apiserver | grep authorization-mode
```

---

> [!tip] CKA Exam
> - The standard authorization mode in kubeadm clusters is `Node,RBAC` — know this
> - **RBAC** is what the exam focuses on — detailed in the next notes
> - Node authorizer specifically handles kubelet API calls — kubelets need certs with `system:node:<nodename>` CN in the `system:nodes` group
> - If you see `AlwaysAllow` on the API server, there's no authorization in place (dangerous in production)
