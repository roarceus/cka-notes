# Helm - Introduction

## The Problem Helm Solves

- A real app (e.g. WordPress) is made of many interconnected Kubernetes objects: a Deployment (web server, database), PersistentVolume + PersistentVolumeClaim, a Service, a Secret for credentials, maybe Jobs for backups, etc.
- Each object typically needs its own YAML file, applied individually with `kubectl apply`
- Customizing defaults (e.g. bumping PV size from 20Gi to 100Gi) means hunting down and editing the right field across multiple files
- Upgrading later means carefully re-editing multiple YAMLs again, and deleting the app means remembering and removing every object individually
- Combining everything into one giant YAML file avoids some of this but makes troubleshooting harder — multiple organized files at least map cleanly to their purpose (e.g. `mydeployment.yaml`)

**The core issue:** Kubernetes has no concept of "an app" — it only sees individual objects and creates each one as declared. It doesn't know that a PV, Deployment, Secret, and Service are all part of one logical "WordPress" application.

---

## What Helm Does

- Helm is a **package manager for Kubernetes** — it understands a group of objects as a single **package** (chart/release), not just a pile of unrelated YAML
- You tell Helm the *package* to act on (e.g. "the WordPress app"), and it figures out which objects that involves — even if there are hundreds
- Analogous to installing a complex program (e.g. a game with thousands of files) via a single installer, instead of manually placing every file yourself

### Key capabilities
- **Install**: a single command creates every object needed for the app
- **Customize**: settings (PV size, site name, admin password, DB config, etc.) live in one place — a `values.yaml` file — instead of scattered across many object YAMLs
- **Upgrade**: a single command updates whatever individual objects need to change
- **Rollback**: Helm tracks revisions/changes over time, so you can roll back to a previous version
- **Uninstall**: a single command removes every object belonging to the app — no need to track them manually

---

> [!tip] CKA Exam
> - Helm = **package manager** (install/uninstall) **+ release manager** (upgrade/rollback) for Kubernetes apps
> - The core value: lets you treat a Kubernetes app as a single unit instead of micromanaging individual objects
> - Custom settings for a chart are centralized in `values.yaml`, not spread across multiple object manifests
