# configuration

Brings the [host](../docs/glossary.md#host) and the [guests](../docs/glossary.md#guest) to their desired
state. **Current tool:** Ansible (one standard project).

> **Status:** not implemented yet.

## Layout

```
configuration/
├── ansible.cfg
├── requirements.yml      # external collections and roles
├── inventory/
│   ├── host.yml          # static: the host
│   ├── guests.incus.yml  # dynamic: guests discovered from Incus
│   └── group_vars/
├── playbooks/
│   ├── host.yml          # OS and Incus on the host (runs before provisioning)
│   ├── database.yml      # PostgreSQL guest
│   └── cluster.yml       # K3s control plane and workers; seeds Argo CD
└── roles/                # own roles, named by the software they configure
```

Inventory groups: `host`, `guests`, `database`, `control_plane`, `workers`, and `cluster`
(children: `control_plane`, `workers`).

`playbooks/cluster.yml` installs Argo CD once and applies the root Application from
[`delivery/`](../delivery/); from then on Argo CD manages itself.

See [ADR-0005](../docs/adr/0005-organize-the-repository-by-lifecycle-stage.md#configuration).
