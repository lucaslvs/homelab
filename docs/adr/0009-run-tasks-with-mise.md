---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Run tasks with mise

## Context and Problem Statement

[ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md) deliberately kept the order of execution
out of directory names and assigned it to a task runner: the host playbook runs before provisioning, the
OpenTofu root modules apply in the order `networks → profiles → guests` (destroy in reverse), and the guest
playbooks run afterwards. Day-to-day commands (format, lint, plan, apply) also need short, stable names.
CI runs many of the same commands, and [ADR-0008](0008-manage-workstation-tool-versions-with-mise.md)
already pins the tools they use with mise.

Which task runner defines these commands and their order, and how do the workstation and CI use it?

## Decision Drivers

* One definition of each command, shared by the workstation and CI (local/CI parity)
* Express dependencies and order between tasks (e.g. apply order of the root modules)
* Tasks run with the tool versions pinned in ADR-0008
* Few tools and files to learn and maintain
* Keep the root of the repository as decided in ADR-0005: stages, `docs/`, and files tools require there
* Readable for portfolio readers

## Considered Options

* mise tasks
* Task (go-task)
* just
* make

## Decision Outcome

Chosen option: "mise tasks", because mise is already the tool manager of the workstation and CI
(ADR-0008), so tasks run with the pinned tool versions without any extra tool, file or CI setup step, and a
single `mise.toml` describes both *which versions* and *which commands* the repository uses.

The decision is applied as follows:

**CI calls tasks**

* CI workflows always run tasks (e.g. `mise run check`), never inline tool commands. The workflow only
  orchestrates (when, on which paths, with which permissions); what a check does is defined once, in the
  task.

**Task names**

* Cross-stage tasks have a plain name: `fmt`, `lint`, `check` (formatting in verify mode plus linters, as
  run by CI on pull requests).
* Tasks that operate one stage are named `<stage>:<action>`, using the stage names of ADR-0005 (e.g.
  `provisioning:plan`, `provisioning:apply`, `provisioning:destroy`, `configuration:host`,
  `configuration:guests`). As with commit scopes ([ADR-0006](0006-define-a-closed-list-of-commit-scopes.md)),
  the stage says *where*, the action says *what*.
* The exact list of tasks grows with the implementation; this ADR fixes the naming rule, not the list.

**Where task code lives**

* Short tasks are defined inline in `mise.toml`.
* When a task that operates a stage outgrows a one-liner, its script lives inside that stage (e.g.
  `provisioning/scripts/apply.sh`) and `mise.toml` points to it, so code stays with the stage it operates.
* Cross-stage tasks that outgrow a one-liner live in `.mise/tasks/`, a hidden directory by mise
  convention, consistent with the root rule of ADR-0005. No visible tasks directory is added at the root.

**Order of execution**

* The order between stages and between root modules is expressed with task dependencies and in the
  task scripts, and documented in the stage READMEs.

### Consequences

* Good, because the workstation and CI run the exact same commands with the exact same tool versions
* Good, because there is no extra tool or configuration file for tasks
* Good, because `mise tasks` lists every available command, with descriptions, from a fresh clone
* Good, because task names mirror the stages and the commit scopes, so the repository has one vocabulary
* Bad, because tool versions and tasks depend on the same tool; replacing mise means replacing both
* Bad, because mise tasks are less widely recognized than `make` or Task by readers of the portfolio
* Neutral, because `mise.toml` can grow large; long scripts move to the stages or to `.mise/tasks/`

### Confirmation

* `mise tasks` lists `fmt`, `lint`, `check` and the `<stage>:<action>` tasks
* CI workflow steps call `mise run <task>` and contain no inline tool commands
* `mise run provisioning:apply` applies the root modules in the order `networks → profiles → guests`
* No visible task directory exists at the repository root

## Pros and Cons of the Options

### mise tasks

* Good, because it reuses the tool manager already decided, on the workstation and in CI
* Good, because tasks support dependencies, per-task working directory, arguments and source-based caching
* Good, because tasks can be inline TOML or standalone scripts
* Bad, because it couples tasks and tool versions to one tool
* Bad, because it is less known than `make` or Task

### Task (go-task)

`Taskfile.yml`, installed through mise.

* Good, because it is popular in DevOps projects and easy to read
* Good, because `includes` allows one Taskfile per stage, and checksums allow skipping unchanged work
* Good, because it is independent of the tool manager
* Bad, because it adds a tool and a file whose role overlaps with mise

### just

`justfile`, a command runner.

* Good, because the syntax is simple and arguments are easy to pass
* Bad, because it has no source-based caching and only simple dependencies; it is a command runner rather
  than a task runner
* Bad, because it adds a tool and a file

### make

`Makefile`.

* Good, because it is preinstalled on macOS and Linux and universally recognized
* Bad, because it is designed to build files: tasks need `.PHONY`, recipes need tabs, and the syntax has
  many pitfalls for commands that produce no files
* Bad, because macOS ships an old GNU Make (3.81)

## More Information

* mise tasks: <https://mise.jdx.dev/tasks/>
* mise file tasks: <https://mise.jdx.dev/tasks/file-tasks.html>
* Task: <https://taskfile.dev/>
* just: <https://just.systems/>
