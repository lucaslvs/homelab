---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Define a closed list of commit scopes

## Context and Problem Statement

[ADR-0003](0003-use-conventional-commits.md) adopted Conventional Commits, where the *type* says what kind
of change a commit is and the optional *scope* says where it happens. It left scopes free-form until the
repository structure was decided, and required the closed list to include `deps` (used by
`chore(deps)`). [ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md) organized the repository by
lifecycle stage (`bootstrap/`, `provisioning/`, `configuration/`, `delivery/`) plus `docs/` and the
tooling files required at the root.

Which scopes are allowed, and how is the list kept in sync with the repository structure?

## Decision Drivers

* Scopes mirror the areas of the repository, so history and changelog can be filtered by area
* A rule simple enough to apply without thinking and to check automatically (linter, CI)
* A list that stays valid as the repository grows, without an ADR for every new workload
* Include `deps`, as required by ADR-0003

## Considered Options

* One scope per top-level area
* One scope per top-level area, with `delivery` split into its layers
* One scope per software or subject

## Decision Outcome

Chosen option: "One scope per top-level area", because it is a mechanical rule (the scope is the name of
the area that changed), it keeps the list closed and short, and it allows automation, such as a future CI
check that the scope matches the changed paths.

**Allowed scopes**

| Scope | Area | Example |
|-------|------|---------|
| `bootstrap` | `bootstrap/` | `docs(bootstrap): document bios settings for lid close` |
| `provisioning` | `provisioning/` | `feat(provisioning): add k3s worker guest` |
| `configuration` | `configuration/` | `fix(configuration): wait for cloud-init before configuring guests` |
| `delivery` | `delivery/` | `feat(delivery): add grafana` |
| `adr` | `docs/adr/` | `docs(adr): add ADR-0005 repository structure` |
| `glossary` | `docs/glossary.md` | `docs(glossary): add delivery stage` |
| `runbooks` | `docs/runbooks/` | `docs(runbooks): add cluster rebuild procedure` |
| `workstation` | Workstation tooling files at the root (e.g. tool versions, git hooks, task runner, editor config) | `build(workstation): add mise config` |
| `deps` | Routine upgrades of software running in the environment (see ADR-0003) | `chore(deps): upgrade k3s to v1.31` |

**Rules**

* The scope is the area that changed. The specific software or subject (e.g. grafana, k3s, a guest)
  goes in the description.
* The scope stays optional: omit it when a change spans several areas (e.g. `style: apply formatters`)
  or touches only root files outside the areas above (e.g. the root `README.md`, `.github/` with type
  `ci`).
* `deps` is used for routine upgrades (`chore(deps)`). An upgrade typed `feat` or `fix` to gain a
  capability or fix a bug (see ADR-0003) uses the scope of the area it changes.
* The rule is the decision; the table is the list it produces today. When a new top-level area is
  created under ADR-0005 (a new stage, or a new entry under `docs/`), its scope is added to the linter
  configuration in the same commit that creates the area. A scope that does not follow the rule requires
  a new ADR.
* The scopes used as examples in ADR-0003 (e.g. `incus`, `ansible`, `k3s`) were illustrative and are
  replaced by this list.

### Consequences

* Good, because every scope maps to exactly one path, so history can be filtered with
  `git log -- <path>` and by scope alike
* Good, because the list grows only when the structure grows, not with every new workload
* Good, because the linter can reject unknown scopes, and CI can later check scope against changed paths
* Bad, because `delivery`, the busiest area, does not distinguish `system`, `platform` and `apps` in the
  changelog; the description carries that detail
* Neutral, because commits made before this ADR keep their scopes and are not rewritten

### Confirmation

* The commit message linter (chosen in the workstation tooling ADR) is configured with exactly the scopes
  above and rejects any other scope
* Optionally, a CI check verifies that a commit's scope matches the paths it changes

## Pros and Cons of the Options

### One scope per top-level area

* Good, because the rule is mechanical: scope = area name
* Good, because the list is short and closed
* Bad, because changes inside `delivery/` are grouped together

### One scope per top-level area, with `delivery` split into its layers

`system`, `platform` and `apps` instead of `delivery`, e.g. `feat(platform): add grafana`.

* Good, because the changelog is more detailed where most changes happen
* Good, because the scopes read well
* Bad, because the rule becomes "first level, except in `delivery/`, where it is the second level"

### One scope per software or subject

`k3s`, `grafana`, `incus`, …

* Good, because scopes are very specific
* Bad, because the list cannot be closed: every new workload adds a scope
* Bad, because some names are ambiguous across areas (e.g. `k3s` is installed in `configuration/` and
  related workloads live in `delivery/`)

## More Information

* Conventional Commits scopes: <https://www.conventionalcommits.org/en/v1.0.0/#specification>
* commitlint `scope-enum` rule: <https://commitlint.js.org/reference/rules.html#scope-enum>
* Repository structure: [ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md)
