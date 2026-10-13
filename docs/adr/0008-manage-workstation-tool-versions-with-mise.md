---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Manage workstation tool versions with mise

## Context and Problem Statement

The homelab is operated from the workstation (see
[ADR-0004](0004-adopt-a-shared-vocabulary-for-machines-and-roles.md)) with many command-line tools:
OpenTofu, Ansible and its linters, kubectl, k9s, Helm, YAML linters, the commit message linter, and more
to come. The same tools run in CI. If the workstation and CI run different versions, a check can pass
locally and fail in CI (e.g. `tofu fmt` output differs between versions), and nobody can tell from the
repository which versions the automation expects.

How are the versions of the command-line tools pinned, installed and kept identical on the workstation and
in CI?

## Decision Drivers

* The same tool versions on the workstation and in CI, from a single file
* Versions recorded in the repository, so upgrading a tool is a reviewable `build` commit (see
  [ADR-0003](0003-use-conventional-commits.md))
* Integrity of downloaded binaries (supply chain)
* One tool for every kind of CLI used here: single binaries (OpenTofu, kubectl, k9s, Helm) and Python
  tools (Ansible, ansible-lint, yamllint)
* Low learning cost, so the focus stays on the homelab itself
* Supported by dependency update bots, so upgrades can be automated later

## Considered Options

* mise
* asdf
* Homebrew with a `Brewfile`
* Nix (directly or through devbox)

## Decision Outcome

Chosen option: "mise", because it is the only option that pins exact versions of both single-binary and
Python tools in one file that the workstation and CI read alike (through the official GitHub Action), with
a lockfile that verifies checksums. The owner already uses mise on the workstation, so it adds no new
tool.

The decision is applied as follows:

* A `mise.toml` file at the repository root (the visible name, the current default in the mise
  documentation) lists every command-line tool used by the automation, the git hooks or CI.
* Versions are **exact** (e.g. `1.11.2`), never ranges or `latest`.
* A `mise.lock` lockfile is committed. It records the download URL and checksum of each tool for the
  platforms in use, `macos-arm64` (workstation) and `linux-x64` (GitHub-hosted runners). CI installs
  with `mise install --locked`, which fails if a tool is not in the lockfile.
* Python tools are installed through mise's Python backends, so no global Python packages are needed on
  the workstation.
* Upgrading a tool means changing `mise.toml` and `mise.lock` in the same commit, typed `build`
  (e.g. `build: upgrade opentofu to 1.11.3`).
* Graphical applications of the workstation (e.g. Headlamp) are out of scope: they do not take part in the
  automation or in CI, and are installed by the owner however they prefer.

### Consequences

* Good, because the workstation and CI run exactly the same versions, from the same file
* Good, because downloaded binaries are verified against committed checksums
* Good, because a fresh clone is ready with `mise install`
* Good, because tool upgrades are explicit, reviewable commits
* Good, because Renovate supports `mise.toml`, so upgrades can be automated later
* Neutral, because mise can also run tasks and set environment variables; whether to use those features is
  decided separately (task runner ADR)
* Bad, because contributors need mise installed before anything else
* Bad, because the lockfile must be refreshed for every platform when a version changes
  (`mise lock --platform macos-arm64,linux-x64`)
* Bad, because graphical tools are not reproducible from the repository

### Confirmation

* `mise.toml` and `mise.lock` exist at the root, with exact versions for every tool used by hooks, tasks and
  CI
* CI installs tools with the mise GitHub Action and `--locked`
* No CI workflow installs a tool by another means (e.g. a separate setup action with its own version)
  unless an ADR allows it

## Pros and Cons of the Options

### mise

* Good, because it installs single binaries (aqua, GitHub releases) and Python tools from one file
* Good, because its lockfile records checksums per platform
* Good, because the official GitHub Action reads the same file in CI
* Good, because it is already installed and used on the workstation
* Neutral, because it also offers tasks and environment variables
* Bad, because it is younger and changes quickly (frequent releases)

### asdf

The predecessor of mise, configured with `.tool-versions`.

* Good, because it is widely known
* Bad, because every tool depends on a community plugin of varying quality
* Bad, because it has no lockfile with checksums
* Bad, because it is slower, and mise reads `.tool-versions` anyway

### Homebrew with a `Brewfile`

* Good, because it is native to macOS and needs nothing new
* Bad, because it does not pin versions: `brew bundle` installs the latest version, so the workstation and
  CI drift apart
* Bad, because it is slow and uncommon on Linux CI runners

### Nix (directly or through devbox)

* Good, because it gives the strongest reproducibility, down to system libraries
* Bad, because Nix is a language and an ecosystem of its own, a large learning detour from the homelab
* Bad, because it is heavy on macOS and in CI

## More Information

* mise: <https://mise.jdx.dev/>
* mise lockfile: <https://mise.jdx.dev/dev-tools/mise-lock.html>
* mise GitHub Action: <https://github.com/jdx/mise-action>
* Renovate mise manager: <https://docs.renovatebot.com/modules/manager/mise/>
* Follow-up decision (separate ADR): automated dependency updates (e.g. Renovate), covering `mise.toml`
  and the other dependencies of the repository
