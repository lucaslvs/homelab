---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Lint commit messages with commitlint

## Context and Problem Statement

[ADR-0003](0003-use-conventional-commits.md) adopted Conventional Commits with a fixed list of types, and
[ADR-0006](0006-define-a-closed-list-of-commit-scopes.md) a closed list of scopes. ADR-0003 requires a
breaking change to carry **both** `!` and a `BREAKING CHANGE:` footer, and left open whether that rule can
be enforced. Enforcement is a local `commit-msg` hook plus a CI check.

[ADR-0007](0007-contribute-through-pull-requests-with-squash-merge.md) made the pull request title and
body the message of the commit that lands on `main`, so CI validates that title and body.
[ADR-0009](0009-run-tasks-with-mise.md) requires CI to call tasks, and
[ADR-0010](0010-manage-git-hooks-with-lefthook.md) runs the `commit-msg` hook with lefthook.

Which tool validates commit messages, and how is it wired into the hook and CI?

## Decision Drivers

* Enforce the types of ADR-0003 and the scopes of ADR-0006 exactly
* Enforce that `!` and the `BREAKING CHANGE:` footer appear together, without custom code
* The same tool and configuration for the local hook and for CI
* Validate an arbitrary message (pull request title and body), not only existing commits
* Widely recognized by readers of the portfolio
* Installable and pinned through mise ([ADR-0008](0008-manage-workstation-tool-versions-with-mise.md))

## Considered Options

* commitlint
* cocogitto
* commitizen
* A GitHub Action that validates only pull request titles

## Decision Outcome

Chosen option: "commitlint", because it is the only option that enforces every rule of ADR-0003 and
ADR-0006 with built-in rules, including `breaking-change-exclamation-mark`, which requires `!` and a
`BREAKING CHANGE:` footer to appear together or not at all. This answers the open question of ADR-0003:
the rule is enforced, not only documented.

The decision is applied as follows:

**Configuration**

* A commitlint configuration file at the repository root, based on `@commitlint/config-conventional`
  (whose types are the ones adopted in ADR-0003), with:
  * `scope-enum` set to exactly the scopes of ADR-0006, at error level (scopes stay optional);
  * `breaking-change-exclamation-mark` enabled at error level.
* commitlint, its configuration package and the Conventional Commits parser it needs to understand `!`
  are pinned through mise, with Node managed by mise. If they cannot be resolved without one, a minimal
  `package.json` with a lockfile is added at the root; this is verified during implementation.
* When a new scope is added under ADR-0006, the configuration changes in the same commit.

**Where it runs**

* Locally, the lefthook `commit-msg` hook runs commitlint on the message file.
* In CI, a task validates the pull request **title and body** as one message (title as header, body as
  body and footers), on the `opened`, `edited`, `synchronize` and `reopened` pull request events, so
  editing the title re-runs the check.
* The pull request title and body reach the task through environment variables, never interpolated
  directly into a shell command, because they are untrusted input (script injection).
* The check is a required status check of the `main` ruleset (ADR-0007).
* Pull requests opened by bots (e.g. dependency updates, release pull requests) must follow the same
  rules and are configured to do so.

**Not done (for now)**

* No re-validation of commits after they land on `main`: the ruleset has no bypass, so every commit on
  `main` comes from a validated pull request.
* No check that the scope matches the changed paths (the optional check of ADR-0006). It is postponed
  until the stages contain code, and would be decided in a new ADR.

### Consequences

* Good, because every rule of ADR-0003 and ADR-0006 is enforced, including the breaking change rule
* Good, because the hook and CI use the same tool and configuration
* Good, because commitlint is the reference tool of the Conventional Commits ecosystem
* Good, because editing a pull request title re-runs the check
* Bad, because it brings Node into the repository tooling, possibly with a `package.json`
* Bad, because it only validates; the changelog needs a separate tool
* Neutral, because adding a scope requires changing the linter configuration, as ADR-0006 already states

### Confirmation

* `git commit -m "update stuff"` is rejected by the hook
* A pull request titled `feat(foo): add x` (unknown scope) fails the check
* A pull request titled `feat(provisioning)!: …` without a `BREAKING CHANGE:` footer in the body fails,
  and so does a body with the footer under a title without `!`
* Fixing the title of a failing pull request re-runs the check, which then passes

## Pros and Cons of the Options

### commitlint

Node.js; configuration based on `@commitlint/config-conventional`.

* Good, because `type-enum`, `scope-enum` and `breaking-change-exclamation-mark` cover every rule
* Good, because it reads a message from standard input, so validating a pull request title and body is
  simple
* Good, because it is the most widely used commit message linter
* Bad, because it needs Node and npm packages (configuration, parser preset)
* Bad, because it only validates messages

### cocogitto

`cog`, a single Rust binary.

* Good, because one tool validates commits, generates the changelog and bumps versions
* Good, because it is a single binary, without Node, and supports a closed list of scopes
* Bad, because no support for requiring `!` and the `BREAKING CHANGE:` footer together was found; it would
  need a custom script
* Bad, because it is less widely known

### commitizen

`cz`, Python.

* Good, because it also validates, generates the changelog and bumps versions
* Bad, because validation is a hand-written regular expression; "`!` if and only if a footer" is fragile
  to express that way
* Bad, because it is centered on an interactive commit flow that is not needed here

### A GitHub Action that validates only pull request titles

E.g. `amannn/action-semantic-pull-request`.

* Good, because it needs no configuration in the repository beyond the workflow
* Bad, because it does not cover the local hook nor the footer in the body
* Bad, because CI would run a tool directly instead of a task, contrary to ADR-0009

## More Information

* commitlint: <https://commitlint.js.org/>
* commitlint rules (`scope-enum`, `breaking-change-exclamation-mark`):
  <https://commitlint.js.org/reference/rules.html>
* cocogitto: <https://docs.cocogitto.io/>
* commitizen: <https://commitizen-tools.github.io/commitizen/>
* GitHub Actions script injection:
  <https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections>
