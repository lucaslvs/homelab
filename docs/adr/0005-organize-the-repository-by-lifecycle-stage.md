---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Organize the repository by lifecycle stage

## Context and Problem Statement

The homelab is automated in layers, each handled today by a different tool: the host is bootstrapped
from bare metal (manual runbook now, Ubuntu autoinstall later), Ansible configures the host and installs
Incus, OpenTofu with the Incus provider creates networks, profiles and guests, Ansible configures the
guests (PostgreSQL, K3s), and Argo CD continuously delivers everything that runs inside Kubernetes. The
repository also doubles as a public portfolio and should support a highly automated, smooth CI/CD flow.

The tools do not map one-to-one to the subjects they manage: a guest is *created* by OpenTofu and
*configured* by Ansible, and Ansible touches the host, the guests and the cluster installation. Any tool
may also be replaced over time (e.g. Ansible by Salt, OpenTofu by Pulumi, Argo CD by Flux).

How is the repository organized: how many repositories, which top-level directories, and where does each
concern live, so that names stay meaningful when tools change?

## Decision Drivers

* Names that describe the *role* of each area, not the tool that currently implements it, so they survive
  tool replacements (in the same way that `system`, `platform` and `apps` describe workloads)
* One consistent rule per directory level, elegant and clear to professional readers
* Each top-level area is the natural project root of one kind of tool (its own config, lint and CI path
  filter), so the structure works *with* the tools
* The whole homelab, from bare metal to applications, readable in one place by portfolio readers
* Smooth, highly automated CI/CD: changes in one area trigger only that area's checks
* Use the shared vocabulary of [ADR-0004](0004-adopt-a-shared-vocabulary-for-machines-and-roles.md)

## Considered Options

* Single public repository, organized by lifecycle stage, then subject ("stage → subject")
* Single public repository, organized by tool (`ansible/`, `opentofu/`, `kubernetes/`)
* Single public repository, organized by numbered layer (`00-bootstrap/`, `10-host/`, `20-guests/`, …)
* Single public repository, with role-named stages and the Kubernetes layers at the root
* Single public repository, organized by subject (`host/`, `guests/`, `cluster/`), stage inside
* Single public repository, with subject nouns aligned to tools (`metal/`, `infrastructure/`,
  `machines/`, `cluster/`)
* Separate repositories for infrastructure and GitOps manifests
* Private repository, with a separate public showcase

## Decision Outcome

Chosen option: "Single public repository, organized by lifecycle stage, then subject", because it is the
only option that names every top-level area by its role, so names survive replacing any tool with another
of the same kind, while each stage still matches one kind of tool and therefore works as that tool's
natural project root. A single public repository keeps the whole homelab readable in one place for the
portfolio, lets a change that crosses stages land in one commit, and lets Argo CD read the repository
without credentials.

### Repository

* One repository, public. Argo CD watches paths inside it.
* **No secret is ever committed in plain text.** The secrets strategy (e.g. SOPS, Sealed Secrets,
  External Secrets) is a separate, future ADR and must be accepted before the first secret is needed.
* Accepted exposure: details such as LAN addresses or domain names are public.

### The rule: stage → subject

* **Level 1 answers *how*:** the lifecycle stage, named by role, never by tool. Each stage corresponds to
  one kind of tool.
* **Level 2 answers *what*:** the subject, named with the vocabulary of ADR-0004 or by workload purpose.
  Inside a stage, the tool's native layout applies (e.g. `inventory/`, `roles/` for Ansible), and the
  vocabulary appears in the names of files, groups and resources.
* The order of execution is **not** encoded in directory names (it is not linear: configuration runs
  before and after provisioning). It lives in a task runner (chosen in [ADR-0009](0009-run-tasks-with-mise.md))
  and in the stage READMEs.

```
homelab/
├── bootstrap/            # bare metal → host reachable over SSH
├── provisioning/         # Incus networks, profiles and guests
├── configuration/        # desired state of the host and guests
├── delivery/             # workloads inside the cluster
├── docs/                 # cross-stage documentation
├── README.md             # includes the stage table below
└── …                     # files that tools require at the root (.github/, mise.toml, task runner…)
```

The root `README.md` must contain a table mapping each stage to its purpose and current tool. It is part
of this decision, because it removes the lookup cost of role-based names:

| Stage | Purpose | Current tool |
|-------|---------|--------------|
| `bootstrap/` | Bare metal → reachable host | Manual runbook |
| `provisioning/` | Incus networks, profiles, guests | OpenTofu |
| `configuration/` | Desired state of host and guests | Ansible |
| `delivery/` | Workloads in the cluster | Argo CD |

General rules:

* Every stage has a `README.md` stating what it does and how to run it.
* The root contains only the stage directories, `docs/`, and files that tools require at the root (by
  their own convention, e.g. `.github/`, `.editorconfig`, `mise.toml`, git hook and task runner
  configuration). There is no `workstation/` directory.
* There are no environment directories (`dev/`, `prod/`) and no `clusters/` directory: the homelab has
  one environment and one cluster. Adding either requires a new ADR.

### `bootstrap/`

```
bootstrap/
├── README.md             # the runbook: firmware settings, OS install, network, first SSH key
└── autoinstall/          # (later) Ubuntu autoinstall files, replacing steps of the runbook
```

The runbook lives in the stage, not in `docs/`: the stage is the role, and its implementation moves from
manual to automated in the same place.

### `provisioning/`

```
provisioning/
├── README.md             # what each root module does and the apply order
├── modules/              # shared child modules, only once real repetition appears
├── networks/             # root module, own state
├── profiles/             # root module, own state
└── guests/               # root module, own state
```

* One OpenTofu root module per subject, each with its own state. Chosen to isolate failures (smaller
  blast radius), to give subjects with different lifecycles independent applies (networks rarely change,
  guests often), and to learn the multi-state pattern used in larger organizations.
* Root modules reference each other's resources **by naming convention** (Incus references networks and
  profiles by name), passed as variables. They do not read each other's state.
* Apply order is `networks → profiles → guests` (destroy in reverse), run by the task runner for now. An
  OpenTofu orchestrator (e.g. Terragrunt, Terramate) may be adopted later through a new ADR.
* A new kind of Incus resource (e.g. storage pools, projects) gets its own root module, named by subject.
* State files are never committed (`*.tfstate` is git-ignored), because state can hold secrets in plain
  text. The state backend is a separate, future ADR.

### `configuration/`

```
configuration/
├── README.md
├── ansible.cfg
├── requirements.yml      # external collections and roles (installed, not committed)
├── inventory/
│   ├── host.yml          # static: the host is not an Incus guest
│   ├── guests.incus.yml  # dynamic: guests discovered from Incus
│   └── group_vars/
├── playbooks/
│   ├── host.yml          # OS and Incus on the host (runs before provisioning)
│   ├── database.yml      # PostgreSQL guest
│   └── cluster.yml       # K3s control plane and workers; seeds Argo CD
└── roles/                # own roles
```

* One standard Ansible project (not an Ansible collection).
* The host is listed statically; guests are discovered from Incus with the
  [`community.general.incus`](https://github.com/ansible-collections/community.general/blob/main/plugins/inventory/incus.py)
  inventory plugin, so Incus is the single source of truth for which guests exist and provisioning and
  configuration stay decoupled.
* Playbooks are named by subject. Roles are named by the software they configure (e.g. `incus`,
  `postgresql`, `k3s`): the "no tool names" rule applies to stages, while a role name states *what* is
  configured, which is the level-2 subject.
* Inventory groups use the ADR-0004 vocabulary, with underscores: `host`, `guests`, `database`,
  `control_plane`, `workers`, and `cluster` (children: `control_plane`, `workers`). Guest hostnames are
  decided in the guest naming ADR.

### `delivery/`

```
delivery/
├── README.md
├── root/                 # root Application, one ApplicationSet and one AppProject per layer
├── system/               # what the cluster needs to work (e.g. argocd/, cert-manager/)
├── platform/             # services that applications consume (e.g. observability)
└── apps/                 # what people use (e.g. home services, own applications)
```

* **Seeding:** `configuration/playbooks/cluster.yml` installs Argo CD once, using the same values kept in
  `delivery/system/argocd/`, and applies the root Application. From then on, Argo CD manages itself; its
  upgrades are commits in `delivery/`. This is the only cross-stage reference and is documented in both
  READMEs.
* **Applications:** one ApplicationSet per layer (`system`, `platform`, `apps`) with a Git directory
  generator: each directory becomes one Application, named after the directory and deployed to the
  namespace of the same name. Adding a directory and pushing is enough to deploy a new workload.
* One AppProject per layer, so permissions differ per layer (e.g. `apps` cannot create cluster-scoped
  resources), and layers can be ordered (`system` before `platform` before `apps`).
* The directory is named `root/` after Argo CD's "root application", and to avoid *bootstrap*, which the
  glossary reserves for the host.
* The format inside each workload directory (Helm wrapper chart, Kustomize or plain manifests) is decided
  later.

### `docs/`

```
docs/
├── adr/                  # architecture decision records
├── glossary.md           # shared vocabulary
└── runbooks/             # Day 2 operations that cross stages (restore a backup, rebuild the cluster…)
```

How a stage works belongs in that stage's README; how to operate the homelab when something happens
belongs in `docs/runbooks/`. Where the architecture overview lives (replacing the transitional
`ARCHITECTURE.md`) is decided later.

### Consequences

* Good, because replacing a tool with another of the same kind changes no directory name
* Good, because each stage is the natural project root of its tool, so lint and CI checks map to one
  path each (e.g. `configuration/**` → `ansible-lint`)
* Good, because the ADR-0004 vocabulary appears where it is unambiguous: files, groups and resources
* Good, because a new workload is deployed by adding a directory, with no extra manifest
* Good, because the whole homelab is visible in one public repository
* Bad, because role-based names are less common than tool names; readers rely on the README stage table
  to find, e.g., the Ansible inventory
* Bad, because a guest's definition is split between `provisioning/guests/` (creation) and
  `configuration/` (configuration)
* Bad, because three OpenTofu states mean three plans/applies, an ordering to maintain, and repeated
  `versions.tf`/`providers.tf`
* Bad, because the repository is public, so a secrets strategy is mandatory before the first secret
* Neutral, because the guest inventory depends on a recent plugin (community.general 12.0.0 or later)
* Neutral, because a tool that covers several stages (e.g. NixOS provisioning and configuring) would blur
  the stage boundaries and require a new ADR

### Confirmation

* The root contains only the stage directories, `docs/`, and files tools require at the root
* The root README contains the stage table, and every stage has a README
* No directory is named after a tool at the stage level
* No `*.tfstate` file and no plain-text secret is ever committed
* Adding a directory under `delivery/{system,platform,apps}/` creates an Application without other changes
* Commit scopes follow this structure (see
  [ADR-0006](0006-define-a-closed-list-of-commit-scopes.md))

## Pros and Cons of the Options

### Single public repository, organized by lifecycle stage, then subject

`bootstrap/`, `provisioning/`, `configuration/`, `delivery/`, `docs/`; subjects inside.

* Good, because names describe roles and survive tool replacements
* Good, because there is one rule per level (level 1: how; level 2: what)
* Good, because each stage matches one kind of tool, so it is that tool's natural project root
* Good, because stage names are standard industry disciplines (provisioning, configuration management,
  continuous delivery)
* Bad, because it is less common than tool names in homelab repositories
* Bad, because `configuration/` is a generic word; the glossary and README give it its precise meaning

### Single public repository, organized by tool

`bootstrap/`, `ansible/`, `opentofu/`, `kubernetes/`, `docs/`. The most common layout among well-known
homelabs (e.g. onedr0p/home-ops, bjw-s-labs/home-ops, ricsanfre/pi-cluster).

* Good, because it is the most recognizable layout; anyone finds `ansible/` instantly
* Good, because each directory is its tool's natural project root
* Bad, because names change whenever a tool is replaced
* Bad, because names say which tool is used, not what role it plays

### Single public repository, organized by numbered layer

`00-bootstrap/`, `10-host/`, `20-guests/`, `30-cluster/`, `40-apps/`.

* Good, because the tree tells the story in execution order
* Bad, because Ansible is split across several layers (several inventories, or shared roles outside the
  layer model)
* Bad, because layers mix tools, so lint and CI need per-directory, per-tool configuration
* Bad, because numbers age badly when a layer is inserted

### Single public repository, with role-named stages and the Kubernetes layers at the root

`bootstrap/`, `configuration/`, `provisioning/`, `system/`, `platform/`, `apps/`, `docs/`. Inspired by
khuedoan/homelab (`metal/`, `system/`, `platform/`, `apps/`), where each layer is handled by a single tool
and there is no guest layer.

* Good, because the Kubernetes layers stand out
* Bad, because the root mixes two criteria (stages and workload layers)
* Bad, because Argo CD and CI watch three root paths instead of one

### Single public repository, organized by subject, stage inside

`host/{bootstrap,configuration}/`, `guests/{provisioning,configuration}/`, `cluster/…`.

* Good, because it uses the ADR-0004 vocabulary at the root
* Bad, because Ansible is split in two projects (two inventories, or shared roles outside the rule)
* Bad, because boundaries are ambiguous (do Incus networks belong to `host/` or `guests/`; does the K3s
  installation belong to `guests/` or `cluster/`?)

### Single public repository, with subject nouns aligned to tools

`metal/`, `infrastructure/`, `machines/`, `cluster/`: the chosen structure with nouns instead of stages.

* Good, because it works technically (same boundaries as the chosen option)
* Bad, because a noun does not tell where to look: a guest is both "infrastructure" (created) and a
  "machine" (configured)
* Bad, because `infrastructure/` is conventionally used for cluster add-ons in Flux repositories

### Separate repositories for infrastructure and GitOps manifests

* Good, because Argo CD only reads the manifests repository, which matters when credentials are scoped
* Good, because each repository could have a different visibility
* Bad, because a change across layers becomes coordinated commits in two repositories
* Bad, because CI, hooks, changelog and ADRs are duplicated or split
* Bad, because the portfolio story is split in two
* Neutral, because Argo CD's advice to separate config from source code is about application source code,
  not about splitting infrastructure

### Private repository, with a separate public showcase

* Good, because mistakes (e.g. a committed secret) are not exposed
* Bad, because the real work is not visible to portfolio readers
* Bad, because the showcase is a second repository that drifts out of date
* Bad, because Argo CD needs credentials to read a private repository

## More Information

* Reference layouts consulted:
  [khuedoan/homelab](https://github.com/khuedoan/homelab),
  [onedr0p/home-ops](https://github.com/onedr0p/home-ops),
  [bjw-s-labs/home-ops](https://github.com/bjw-s-labs/home-ops),
  [ricsanfre/pi-cluster](https://github.com/ricsanfre/pi-cluster),
  [fluxcd/flux2-kustomize-helm-example](https://github.com/fluxcd/flux2-kustomize-helm-example)
* Ansible sample setup: <https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html>
* Argo CD ApplicationSet Git generator:
  <https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Git/>
* Argo CD cluster bootstrapping (app of apps):
  <https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/>
* Argo CD best practices (config vs source code repositories):
  <https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/>
* Follow-up decisions (separate ADRs): secrets strategy; OpenTofu state backend; workstation tooling,
  including the task runner (see [ADR-0008](0008-manage-workstation-tool-versions-with-mise.md),
  [ADR-0009](0009-run-tasks-with-mise.md) and [ADR-0010](0010-manage-git-hooks-with-lefthook.md)); format
  of workload directories in `delivery/`; location of the architecture overview; guest naming scheme
