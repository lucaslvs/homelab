---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Adopt a shared vocabulary for machines and roles

## Context and Problem Statement

The homelab stacks several layers (physical machine, Incus, guests, Kubernetes) and several tools, each
with its own vocabulary. Many everyday words already have a meaning in at least one of these tools:
`server` is the K3s control plane, `node` is a Kubernetes node, `host` is any machine in an Ansible
inventory, `instance` is anything Incus runs, and `master` is deprecated in Kubernetes. These names will
end up in directories, inventory groups, resource names, scopes and docs.

Which names do we use for the machines and roles of the homelab, so that each word means one thing?

## Decision Drivers

* One meaning per term across docs, code and commit scopes
* Avoid collisions with terms owned by the tools in use
* Terms that stay valid if a tool is replaced (e.g. another Kubernetes distribution, LXC vs VMs)
* Widely recognized, current industry terminology, so the repository reads as professional work to
  readers of the portfolio

## Considered Options

* Role-based vocabulary: workstation, host, guest, control plane / worker
* Tool-native vocabulary: control node (Ansible), server (common usage), instance (Incus), server / agent (K3s)
* No formal vocabulary

## Decision Outcome

Chosen option: "Role-based vocabulary", because it uses current industry terminology that readers of the
portfolio recognize, and it names what each machine *is* in the architecture rather than what one tool
calls it, which avoids the `server`/`node` collisions and survives tool changes.
Tool-native names are kept as mappings in [the glossary](../glossary.md).

| Term | Means | Rejected alternatives |
|------|-------|-----------------------|
| **workstation** | The machine from which the homelab is operated with client tools; currently the owner's Mac | control node (Ansible-only), operator machine (uncommon) |
| **host** | A physical machine that runs Incus and hosts guests; currently the Dell notebook | server (collides with K3s), node (collides with Kubernetes), hypervisor (names the software, not the machine) |
| **guest** | Any Incus instance on a host, system container or VM | instance (ambiguous outside Incus), VM / container (no collective term) |
| **control plane** / **worker** | Roles of Kubernetes nodes | server / agent (K3s-specific), master / worker (deprecated) |

### Consequences

* Good, because each term has a single meaning in this repository
* Good, because names stay valid if K3s, Incus, the LXC/VM choice or the hardware changes
* Bad, because code still uses tool-native names (`incus_instance`, `k3s server`), so readers need the
  glossary mapping
* Neutral, because `host` still overlaps with Ansible's generic "host" (any managed node in the
  inventory); inventory groups and the glossary disambiguate it

### Confirmation

* New directories, inventory groups, resource names, scopes and docs use these terms
* New terms that need a shared meaning are added to the [glossary](../glossary.md); terms that involve a
  trade-off get an ADR

## More Information

* Glossary: [docs/glossary.md](../glossary.md)
* Kubernetes removed "master" in favor of "control-plane":
  <https://kubernetes.io/docs/reference/labels-annotations-taints/#node-role-kubernetes-io-control-plane>
* K3s architecture (server / agent): <https://docs.k3s.io/architecture>
