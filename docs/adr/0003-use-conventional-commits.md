---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Use Conventional Commits

## Context and Problem Statement

The Git history of this repository is part of the portfolio: it should tell how the homelab evolved, layer
by layer. Commits are made by the owner, often prepared with the help of AI assistants, which makes a
shared, explicit message convention even more important. Free-form messages make the history hard to
scan, filter and turn into a changelog.

Which commit message convention should the repository follow, and how should it be enforced?

## Decision Drivers

* A readable, scannable history that shows the evolution of the project
* Ability to filter history by kind of change and by area of the repository
* Automatic changelog generation from the history
* Rules that hold for every commit, whether written by hand or prepared with AI assistance, without
  relying on memory
* Low friction for day-to-day commits from the workstation

## Considered Options

* Conventional Commits 1.0.0
* Free-form messages with a short style guide
* gitmoji

## Decision Outcome

Chosen option: "Conventional Commits 1.0.0", because it is the only option that is both
**machine-checkable** (so the same rules can be enforced for every commit, whether written by hand or
prepared with AI assistance, and the history can drive filtering and changelog generation) and **widely
recognized** (so readers of the portfolio understand the history without learning a custom convention). It
costs some friction per commit, which this ADR keeps small with a per-type guide for this repository.

The convention is applied as follows:

**Format**

```
<type>(<scope>)!: <description>

[optional body]

[optional footer(s)]
```

* `description` is in English (see [ADR-0002](0002-use-english-as-repository-language.md)), imperative
  mood, lowercase, no trailing period
* Each commit contains one logical change; if it needs more than one type, split it. Documentation of the
  change itself (e.g. updating a README when adding a service) belongs in the same commit
* A change is breaking when it cannot be applied as a routine run of the automation, that is, when
  applying it:
  * requires manual steps outside the automation (e.g. moving state, creating a secret by hand, changing
    how the bootstrap is run), or
  * risks losing data or state (e.g. recreating a guest that holds data, changing the cluster datastore)

  Recreating stateless resources that the automation rebuilds on its own is not breaking.
* A breaking change is marked with **both** `!` after the type/scope and a `BREAKING CHANGE:` footer that
  describes what must be done to apply it:

  ```
  feat(k3s)!: move cluster datastore to embedded etcd

  BREAKING CHANGE: back up the sqlite datastore before applying;
  the cluster must be rebuilt from the backup.
  ```
* Footers may reference decisions, e.g. `Refs: ADR-0003`
* The commit author is always the owner, who reviews and is accountable for every change. AI assistance is
  not recorded per commit (no `Co-Authored-By` or similar trailers); the use of AI in this project is
  disclosed at the process level (see [ADR-0001](0001-record-architecture-decisions-using-madr.md))

**Types**

Only the standard types of `@commitlint/config-conventional` are allowed; no custom types (such as `ops`
or `infra`) are added. The *where* of a change is expressed by the scope, not by the type.

| Type | Use in this repository |
|------|------------------------|
| `feat` | A new capability of the environment (new service, new instance, new automation) |
| `fix` | Corrects wrong behavior of the environment or its automation |
| `docs` | Documentation and ADRs only |
| `refactor` | Restructures code/config without changing the resulting environment |
| `perf` | Improves resource usage or speed (RAM, CPU, IOPS) |
| `test` | Adds or changes tests and validations |
| `build` | Tooling of the repository itself, which does not change the running environment (OpenTofu and provider versions, linters, Brewfile, lock files) |
| `ci` | CI pipelines and workflows |
| `style` | Formatting only, no semantic change |
| `chore` | Maintenance that fits nothing above, including routine upgrades of software running in the environment (`chore(deps)`) |
| `revert` | Reverts a previous commit |

Version upgrades are typed by what they affect:

* Repository tooling (e.g. OpenTofu, an OpenTofu provider, `ansible-lint`): `build`
* Software running in the environment (e.g. K3s, an Argo CD chart): `chore(deps)` by default; `feat` or
  `fix` when the upgrade is made to gain a capability or to fix a bug, so it stands out in the changelog

**Examples**

The scopes below are illustrative; they are not the closed list of scopes (see *Scopes*).

```
feat(bootstrap): configure ssh key access on the host
feat(incus): add k3s server guest
fix(ansible): wait for cloud-init before configuring guests
refactor(ansible): extract common role for guest hardening
chore(deps): upgrade k3s to v1.31
build: upgrade incus opentofu provider
style: apply tofu fmt
docs(adr): add ADR-0004 git hook manager
```

| Don't | Why | Do |
|-------|-----|----|
| `update stuff` | No type, says nothing | `fix(k3s): ...` with the actual change |
| `feat: Added K3s.` | Past tense, capitalized, trailing period | `feat(incus): add k3s server guest` |
| `feat: add grafana and fix k3s upgrade` | Two logical changes | Two commits: a `feat` and a `fix` |
| `build: upgrade k3s to v1.31` | K3s runs in the environment, not repository tooling | `chore(deps): upgrade k3s to v1.31` |

**Scopes**

Scopes are optional and free-form for now. A closed list of allowed scopes will be defined once the
repository structure is decided, since scopes are expected to mirror its areas/layers. That decision will
update this convention and the linter configuration.

**Enforcement**

* A local `commit-msg` hook on the workstation rejects non-conforming messages before the commit exists
* A CI check on GitHub validates commit messages on push and pull requests, covering commits made with
  `--no-verify` or from environments without the hook

**Changelog**

A `CHANGELOG.md` is generated from the commit history.

### Consequences

* Good, because the history is uniform and filterable by type and scope
* Good, because a changelog can be generated with no extra writing effort
* Good, because the rules are enforced for every commit, including those prepared with AI assistance
* Bad, because the hook and the CI check add tooling that must be installed and maintained
* Bad, because each commit takes slightly more thought (type, scope, breaking or not)
* Neutral, because commits made before this ADR (e.g. `Initial commit`) are not rewritten and are exempt
* Neutral, because until the enforcement tooling exists, compliance relies on discipline
* Neutral, because AI assistance is not visible per commit; tools that add attribution trailers by default
  must be configured not to

### Confirmation

* The `commit-msg` hook rejects a message such as `update stuff`
* The CI check fails on a pushed commit that does not follow the format
* `CHANGELOG.md` is regenerated from the history without manual edits

## Pros and Cons of the Options

### Conventional Commits 1.0.0

* Good, because it is a widely adopted, documented specification
* Good, because of the rich tooling ecosystem (commitlint, git-cliff, release-please)
* Bad, because the strict format adds some friction

### Free-form messages with a short style guide

* Good, because there is no friction and no tooling
* Bad, because consistency depends on discipline and cannot be linted
* Bad, because the history cannot drive changelog generation

### gitmoji

* Good, because it is visually expressive
* Neutral, because it can also be linted with dedicated tooling; its weakness is recognition, not
  machine-checkability
* Bad, because it is less common in professional/corporate settings
* Bad, because emoji are harder to type and grep from a keyboard-first workflow

## More Information

* Specification: <https://www.conventionalcommits.org/en/v1.0.0/>
* Types reference: <https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional>
* Follow-up decisions (separate ADRs):
  * Git hook manager for the workstation (it will also run formatters/linters such as `tofu fmt`,
    `ansible-lint`, `yamllint`)
  * Commit message linter and CI implementation, including whether `!` can be enforced to require a
    `BREAKING CHANGE:` footer
  * Changelog generator (e.g. git-cliff, release-please) and whether to tag versions/milestones
  * Closed list of scopes, after the repository structure ADR, which must include `deps` (used by
    `chore(deps)`)
  * Contribution workflow (direct pushes vs pull requests, merge strategy), which determines whether CI
    lints individual commits or pull request titles
