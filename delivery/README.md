# delivery

Continuously applies the desired state of the workloads inside the cluster from this repository
(GitOps). **Current tool:** Argo CD.

> **Status:** not implemented yet.

## Layout

```
delivery/
├── root/                 # root Application, one ApplicationSet and one AppProject per layer
├── system/               # what the cluster needs to work (e.g. argocd/, cert-manager/)
├── platform/             # services that applications consume (e.g. observability)
└── apps/                 # what people use (e.g. home services, own applications)
```

* Each directory under `system/`, `platform/` and `apps/` becomes one Application, named after the
  directory and deployed to the namespace of the same name. Adding a directory and pushing deploys it.
* Argo CD is seeded once by
  [`configuration/playbooks/cluster.yml`](../configuration/), using the values in `system/argocd/`;
  afterwards it manages itself.
* The format inside each workload directory (Helm, Kustomize or plain manifests) is a pending decision.

See [ADR-0005](../docs/adr/0005-organize-the-repository-by-lifecycle-stage.md#delivery).
