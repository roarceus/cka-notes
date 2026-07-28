# Deployment with kubeadm - Provision VMs with Vagrant

## Tools Used

- **VirtualBox** — the hypervisor that actually runs the VMs
- **Vagrant** — automation tool that spins up a set of VMs from a single config file, so everyone gets the same reproducible setup with one command
- Prerequisites: install both VirtualBox and Vagrant (via their respective official sites/docs)

---

## The Vagrantfile

- Defines the VM specs: **1 master node + 2 worker nodes**
- VM network uses the `192.168.56.x` range (this is the VM/node network — unrelated to Kubernetes' own pod/service networking)
- Provided in the course repo — clone it to get the Vagrantfile:

```bash
git clone <repo-url>
cd <cloned-folder>
ls
# Vagrantfile should be here
```

---

## Bringing Up the VMs

Check current VM status (all start as `not created`):
```bash
vagrant status
```

Provision all VMs defined in the Vagrantfile (pulls the base image, e.g. Ubuntu Bionic 64, then provisions each VM in order — master first, then worker nodes):
```bash
vagrant up
```

Confirm all nodes are running:
```bash
vagrant status
# kubemaster, kubenode01, kubenode02 → running
```

---

## Connecting to the VMs

```bash
vagrant ssh <node-name>
# e.g.
vagrant ssh kubemaster
vagrant ssh kubenode01
vagrant ssh kubenode02
```

- `logout` exits the SSH session back to the host machine

---

> [!tip] CKA Exam
> - Vagrant + VirtualBox is a common way to reproduce a local multi-node lab environment — not exam-tested directly, but useful for practicing kubeadm setup
> - `vagrant up` provisions VMs; `vagrant ssh <name>` connects to a specific node; `vagrant status` shows current state
> - The VM network IPs (e.g. `192.168.56.x`) are just host-level networking — the actual Kubernetes pod network is configured separately, later, via a CNI plugin
