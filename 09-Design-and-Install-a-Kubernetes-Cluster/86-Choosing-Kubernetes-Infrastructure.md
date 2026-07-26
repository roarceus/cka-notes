# Choosing Kubernetes Infrastructure

## Local Deployment

- On Linux, you can install Kubernetes binaries manually, but an automated tool is generally preferred
- On Windows, native Kubernetes binaries aren't available — you need a virtualization platform (Hyper-V, VMware Workstation, VirtualBox) to run Linux VMs that host Kubernetes

### Minikube vs. Kubeadm

| | Minikube | Kubeadm |
|---|---|---|
| Cluster type | Single-node only | Single-node or multi-node |
| VM provisioning | Automatic (creates/manages VMs itself, e.g. via VirtualBox) | Requires VMs to already be provisioned |
| Best for | Learning, quick local testing | Local or production clusters where you control the VMs |

![Minikube vs Kubeadm comparison](https://kodekloud.com/kk-media/image/upload/v1752869755/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Choosing-Kubernetes-Infrastructure/frame_140.jpg)

---

## Production Deployment: Turnkey vs. Hosted

| | Turnkey Solutions | Hosted (Managed) Solutions |
|---|---|---|
| Provisioning | Automated via tools/scripts | Provider manages it entirely |
| VM maintenance/patching/upgrades | Your responsibility | Provider's responsibility |
| Example | kops on AWS | GKE (deploy in minutes, minimal manual work) |

![Turnkey vs Hosted solutions comparison](https://kodekloud.com/kk-media/image/upload/v1752869757/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Choosing-Kubernetes-Infrastructure/frame_190.jpg)

### Turnkey options (on-prem)
- **OpenShift** (Red Hat) — adds management tooling, GUI, CI/CD integration
- **Cloud Foundry Container Runtime** — open source, uses BOSH to deploy/manage HA clusters
- **VMware Cloud PKS** — deploy Kubernetes on an existing VMware environment
- **Vagrant** — scripts to deploy Kubernetes across various cloud providers

### Hosted (managed) options
- **Google Container Engine (GKE)** — GCP
- **OpenShift Online** — Red Hat
- **Azure Kubernetes Service (AKS)** — Azure
- **Amazon EKS** — AWS

---

> [!tip] CKA Exam
> - **Minikube** = single-node only, provisions its own VM. **Kubeadm** = single or multi-node, but VMs must already exist
> - **Turnkey** = you automate provisioning but still maintain the VMs. **Hosted/managed** = the provider handles infrastructure end-to-end
> - Know a couple of examples in each category (kops = turnkey/AWS; GKE, AKS, EKS = hosted) — exact vendor list isn't critical, but the turnkey-vs-hosted distinction is
