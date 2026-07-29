# Helm - Installation and Configuration

## Prerequisites

- A working Kubernetes cluster
- `kubectl` properly configured, with a valid kubeconfig containing correct cluster credentials — Helm relies on this to connect to the cluster

---

## Installing Helm on Linux

### Snap
```bash
sudo snap install helm --classic
```
- `--classic` gives Helm a more relaxed sandbox so it can access your kubeconfig in your home directory

### APT (Debian/Ubuntu)
```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### PKG
```bash
pkg install helm
```

---

> [!tip] CKA Exam
> - Helm needs a valid, working kubeconfig to function — it uses the same cluster access as `kubectl`
> - Know that Helm is a **package manager for Kubernetes** (manages "charts") and has multiple install paths depending on the OS/package manager
