# Kustomize - Installation and Setup

## Prerequisites
- A running Kubernetes cluster with `kubectl` installed and configured
- Kustomize supports Linux, Windows, and macOS

## Installing

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
```

Verify:
```bash
kustomize version --short
# {kustomize/v4.4.1  2021-11-11T23:36:27Z }
```

- If the version doesn't show up, try reopening your terminal (env vars may not have refreshed) before rerunning the install script

> [!tip] CKA Exam
> Kustomize also ships **built into `kubectl`** (`kubectl apply -k`) — installing the standalone binary is only needed to get a newer version than what's bundled.
