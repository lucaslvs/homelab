# Glossary

Shared vocabulary of this homelab. Terms in **Machines and roles** are decided in
[ADR-0004](adr/0004-adopt-a-shared-vocabulary-for-machines-and-roles.md); the rest describe how
common terms are used here. When a new term needs a shared meaning, add it here; when choosing it involves
a trade-off, record an ADR as well.

## Machines and roles

### Workstation

The machine from which the homelab is operated; currently the owner's Mac. Runs the client tools,
e.g. OpenTofu, Ansible, kubectl, k9s, Headlamp. Ansible calls it the *control node*.

### Host

A physical machine that runs Incus and hosts guests; currently the Dell notebook, running Ubuntu
Server. Not to be called *server* or *node*.

### Guest

Any Incus instance running on a host, whether a system container (LXC) or a virtual machine. Incus
and its OpenTofu provider call it an *instance* (`incus_instance`).

### Control plane

The Kubernetes node role that runs the API server and cluster management components. K3s calls it a
*server* node. Do not use *master*.

### Worker

The Kubernetes node role that runs application workloads. K3s calls it an *agent* node.

## Kubernetes

### Node

A machine that is a member of the Kubernetes cluster, in the control plane or worker role. In the
current design, nodes are guests.

### Cluster

The K3s Kubernetes cluster formed by the control plane and worker nodes. Not to be confused with an
Incus cluster (several hosts).

## Lifecycle

### Host bootstrap

Taking a host from bare metal to reachable over SSH with key-based access: firmware (BIOS/UEFI)
settings, OS installation, network and the first SSH key. From there, configuration management takes
over. Not to be confused with bootstrapping the Kubernetes cluster or Argo CD.

### Configuration management

Bringing a reachable machine (host or guest) to its desired state, e.g. with Ansible.

### Provisioning

Creating infrastructure resources declaratively, e.g. with OpenTofu: Incus networks, profiles and
guests.

### Delivery

Continuously applying the desired state of workloads inside the cluster from Git, e.g. with Argo CD
(GitOps). The `delivery/` stage of the repository.

### Day 0 / Day 1 / Day 2

Lifecycle stages as commonly used in operations: design and planning (Day 0); installation, deployment
and initial configuration, including host bootstrap (Day 1); ongoing operation, such as monitoring,
upgrades, backups and recovery (Day 2).

## Repository

### Stage

A top-level area of the repository, named by its role in the lifecycle (bootstrap, provisioning,
configuration, delivery), not by the tool that implements it. See
[ADR-0005](adr/0005-organize-the-repository-by-lifecycle-stage.md).

## Term mapping across tools

| Our term | Ansible | Incus / OpenTofu | K3s | Kubernetes |
|----------|---------|------------------|-----|------------|
| workstation | control node | client (`incus` CLI) | — | client (kubectl) |
| host | managed node (inventory host) | server / cluster member | — | — |
| guest | managed node (inventory host) | instance (`incus_instance`) | — | node (when it runs K3s) |
| control plane | — | — | server | control-plane node |
| worker | — | — | agent | worker node |

## Avoided terms

| Term | Why | Use instead |
|------|-----|-------------|
| master | Deprecated in Kubernetes | control plane |
| server (for a host) | Means the control plane in K3s | host |
| node (for a host) | Means a Kubernetes node | host |
| hypervisor (for a host) | Names the software, not the machine | host |
| instance (outside Incus code) | Ambiguous outside Incus | guest |
| VM / container (as a collective term) | Each names only one kind of guest | guest |
| control node (outside Ansible) | Ansible-specific | workstation |
| server / agent (outside K3s) | K3s-specific | control plane / worker |
