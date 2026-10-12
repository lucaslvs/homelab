# homelab

A bare-metal DevOps homelab: an old notebook turned into a small private cloud, built and operated
declaratively from this repository, from firmware settings to the applications running in Kubernetes.

> **Status:** foundations. Decisions are being recorded; nothing is provisioned yet.

## Stages

The repository is organized by lifecycle stage, named by role rather than by tool
([ADR-0005](docs/adr/0005-organize-the-repository-by-lifecycle-stage.md)).

| Stage | Purpose | Current tool |
|-------|---------|--------------|
| [`bootstrap/`](bootstrap/) | Bare metal → reachable host | Manual runbook |
| [`provisioning/`](provisioning/) | Incus networks, profiles, guests | OpenTofu |
| [`configuration/`](configuration/) | Desired state of host and guests | Ansible |
| [`delivery/`](delivery/) | Workloads in the cluster | Argo CD |

## Documentation

* [Architecture decision records](docs/adr/)
* [Glossary](docs/glossary.md): the shared vocabulary (workstation, host, guest, control plane, worker…)
* [Runbooks](docs/runbooks/): Day 2 operations that cross stages

## Conventions

* Every significant decision is an ADR ([MADR](docs/adr/0001-record-architecture-decisions-using-madr.md))
* Repository content is in English ([ADR-0002](docs/adr/0002-use-english-as-repository-language.md))
* Commits follow [Conventional Commits](docs/adr/0003-use-conventional-commits.md) with a
  [closed list of scopes](docs/adr/0006-define-a-closed-list-of-commit-scopes.md)
* No secret is ever committed in plain text, and no OpenTofu state is committed
