# Configuring Scheduler Profiles

## Scheduling Phases

Every pod goes through these phases before being placed on a node:

| Phase | What happens | Key plugins |
|---|---|---|
| **Queue** | Pod waits; sorted by priority | `PrioritySort` |
| **Filter** | Nodes that can't run the pod are eliminated | `NodeResourcesFit`, `NodeName`, `NodeUnschedulable` |
| **Score** | Remaining nodes scored 0–10; highest wins | `NodeResourcesFit`, `ImageLocality` |
| **Bind** | Pod is assigned to the winning node | `DefaultBinder` |

![Scheduler extension points](https://kodekloud.com/kk-media/image/upload/v1752869885/notes-assets/images/CKA-Certification-Course-Certified-Kubernetes-Administrator-Configuring-Scheduler-Profiles/frame_380.jpg)

**Notable filter-phase behaviours:**
- `NodeName` plugin — honours `nodeName` in pod spec
- `NodeUnschedulable` — excludes nodes marked unschedulable via `kubectl drain` or `kubectl cordon`

---

## Multiple Profiles in One Scheduler Binary

Since **Kubernetes 1.18**, multiple scheduler profiles can run within a single scheduler binary — no need for separate processes. Each profile behaves like an independent scheduler.

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler

  - schedulerName: my-scheduler-2
    plugins:
      score:
        disabled:
          - name: TaintToleration   # turn off a plugin
        enabled:
          - name: MyCustomPluginA   # add a custom plugin

  - schedulerName: my-scheduler-3
    plugins:
      preScore:
        disabled:
          - name: '*'   # disable ALL preScore plugins
      score:
        disabled:
          - name: '*'   # disable ALL score plugins
```

Advantages over separate binaries:
- No race conditions between schedulers competing for the same node
- Simpler to manage — one binary, one config file

---

> [!tip] CKA Exam
> - Know the 4 scheduling phases: **Queue → Filter → Score → Bind**
> - `kubectl cordon <node>` marks a node unschedulable (sets taint + flag) — the `NodeUnschedulable` plugin then filters it out
> - Multiple profiles = one scheduler binary, multiple named schedulers — assign pods via `schedulerName` as usual
> - Deep plugin customisation is unlikely to be tested directly — focus on understanding the phases and what each does
