---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Manage git hooks with lefthook

## Context and Problem Statement

[ADR-0003](0003-use-conventional-commits.md) requires a local `commit-msg` hook that rejects
non-conforming commit messages, and the repository will soon need formatters and linters (e.g. `tofu fmt`,
`ansible-lint`, `yamllint`) before each commit. Git hooks live in `.git/hooks/`, which is not versioned,
so a hook manager is needed to keep the hook configuration in the repository and install it.

With [ADR-0007](0007-contribute-through-pull-requests-with-squash-merge.md), CI on pull requests is the
gate that protects `main`; local hooks are fast feedback, catching mistakes in seconds on the workstation
instead of minutes after a push. Tool versions are pinned by mise
([ADR-0008](0008-manage-workstation-tool-versions-with-mise.md)) and commands are defined as mise tasks
([ADR-0009](0009-run-tasks-with-mise.md)).

Which tool manages the git hooks, and how does it fit with mise?

## Decision Drivers

* Fast feedback before commits, not a second gate (CI is the gate)
* mise stays the single source of tool versions; the hook manager must not pin its own
* One definition of each command (tasks), reused by hooks and CI
* Hooks active right after cloning and installing tools, without a step to remember
* Simple, familiar configuration format
* Maturity and adoption

## Considered Options

* lefthook
* pre-commit
* hk
* `core.hooksPath` with own shell scripts

## Decision Outcome

Chosen option: "lefthook", because it only triggers commands, so tool versions stay in mise and commands
stay in tasks, and it does so with a simple YAML configuration and a mature, widely adopted tool.
hk was a close alternative (see its pros and cons) and may be reconsidered later through a new ADR.

The decision is applied as follows:

* A `lefthook.yml` file at the repository root configures the hooks. lefthook itself is pinned in
  `mise.toml` like every other tool.
* The roles are split: **mise** pins versions, **tasks** define commands, **lefthook** decides *when* they
  run. Hooks run tools through mise (tasks or `mise x`), never tools found elsewhere on the `PATH`.
* `pre-commit` runs formatters and linters on **staged files only**, for speed; checking the whole
  repository is the job of CI (`mise run check`).
* `commit-msg` runs the commit message linter on the message file.
* Files fixed by formatters are re-staged (`stage_fixed`). Before implementing, how lefthook handles
  partially staged files must be verified, so that unstaged work is never committed.
* Hooks are installed **automatically**: a mise `postinstall` hook runs `lefthook install`, so cloning
  the repository and running `mise install` is enough.
* Hooks can be skipped (`git commit --no-verify`); this is accepted, because CI validates every pull
  request regardless.

### Consequences

* Good, because hooks, CI and manual runs share the same tool versions and commands
* Good, because a fresh clone gets working hooks with `mise install`, without an extra step
* Good, because staged-only checks keep commits fast
* Good, because the configuration is plain YAML
* Bad, because it is one more tool and one more root file (allowed by
  [ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md))
* Bad, because lefthook has no built-in definitions of linters; each command is written by hand (as a task)
* Neutral, because the handling of partially staged files must be confirmed during implementation

### Confirmation

* `lefthook.yml` exists at the root and lefthook is pinned in `mise.toml`
* After `mise install` in a fresh clone, `.git/hooks/pre-commit` and `.git/hooks/commit-msg` are installed
  by lefthook
* A commit with the message `update stuff` is rejected locally
* Hook commands call mise tasks or `mise x`, never unpinned tools

## Pros and Cons of the Options

### lefthook

`lefthook.yml`; a single Go binary.

* Good, because it only runs commands and leaves tool versions to mise
* Good, because it supports globs, staged files, parallel commands, re-staging fixed files and
  `commit-msg`
* Good, because it is fast, mature and widely adopted
* Good, because YAML is already familiar
* Bad, because there are no built-in linter definitions
* Neutral, because its handling of partially staged files was not confirmed when this ADR was written

### pre-commit

`.pre-commit-config.yaml`; a Python framework.

* Good, because it is the best-known hook framework, with a large catalogue of ready-made hooks
* Good, because it stashes unstaged changes, handling partially staged files
* Bad, because each hook pins its own tool version (`rev:`) in its own environment, a second source of
  versions next to `mise.toml`; calling tasks as `system` hooks avoids this but gives up its main strength
* Bad, because the first run is slow while it builds hook environments

### hk

`hk.pkl`; by the author of mise.

* Good, because it is designed to work with mise
* Good, because each step declares separate `check` and `fix` commands, so the same definition serves the
  hook and CI
* Good, because it coordinates file locks between fixers and stashes unstaged changes
* Bad, because its configuration language, Pkl, is new to the owner
* Bad, because it is younger and less adopted, with fewer references
* Neutral, because it overlaps with mise tasks as the place where checks are defined

### `core.hooksPath` with own shell scripts

* Good, because it has no dependency and teaches how hooks work internally
* Bad, because staged-file handling, parallelism, re-staging and partially staged files must be written and
  maintained by hand
* Neutral, because it also needs a per-clone setup step (`git config core.hooksPath`)

## More Information

* lefthook: <https://lefthook.dev/>
* pre-commit: <https://pre-commit.com/>
* hk: <https://hk.jdx.dev/>
* mise hooks (`postinstall`): <https://mise.jdx.dev/hooks.html>
* Git hooks: <https://git-scm.com/docs/githooks>
