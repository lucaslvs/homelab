---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Contribute through pull requests with squash merge

## Context and Problem Statement

[ADR-0003](0003-use-conventional-commits.md) adopted Conventional Commits, enforced by a local `commit-msg`
hook and a CI check, and left open whether changes reach `main` by direct push or through pull requests.
In this repository, `main` is not only the published history: under
[ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md), Argo CD watches it, so what lands on `main`
is applied to the cluster. The owner is the only contributor, and the goal is a highly automated, smooth
CI/CD flow.

How do changes reach `main`, how are they merged, and how is `main` protected, so that CI runs *before* a
change becomes the desired state of the homelab?

## Decision Drivers

* `main` is the desired state of the homelab; it must only receive changes that passed CI
* A single, simple target for commit message validation and changelog generation
* One logical change per commit on `main`, as required by ADR-0003
* Room for future automation that works on pull requests (e.g. `tofu plan` comments, dependency update
  bots, release pull requests)
* Low friction for a single contributor
* A professional, recognizable workflow for readers of the portfolio

## Considered Options

* Direct pushes to `main`
* Pull requests with squash merge
* Pull requests with rebase merge
* Pull requests with merge commits

## Decision Outcome

Chosen option: "Pull requests with squash merge", because it is the only option where CI gates every
change before it reaches `main` while message validation stays a single check: the pull request title and
body become the one commit that lands on `main`, so "one pull request = one logical change" maps directly to
ADR-0003's "one logical change per commit".

The workflow is applied as follows:

**Pull requests**

* Every change to `main` goes through a pull request, including the owner's own changes.
* A pull request contains one logical change. Its **title** follows Conventional Commits (type, scope from
  [ADR-0006](0006-define-a-closed-list-of-commit-scopes.md), description), because it becomes the commit
  subject on `main`.
* A breaking change carries `!` in the title and the `BREAKING CHANGE:` footer in the pull request
  **body**, because the body becomes the commit body on `main`.
* Commits inside a branch are discarded by the squash; they still pass the local `commit-msg` hook of
  ADR-0003, but CI validates the pull request title and body, not each branch commit.
* Branch names are free-form.

**Repository settings**

* Only squash merge is allowed; merge commits and rebase merge are disabled.
* The squash commit uses the pull request title as subject (`PR_TITLE`, not the default
  `COMMIT_OR_PR_TITLE`, which would use a single commit's message instead of the validated title) and the
  pull request body as message (`PR_BODY`).
* Head branches are deleted automatically after merge.
* Auto-merge is allowed, so a pull request merges by itself once CI is green.

**Protection of `main`**

A repository ruleset targeting `main`:

* requires a pull request before merging, with zero required approvals (a single owner cannot approve
  their own pull request);
* requires the CI status checks to pass;
* requires linear history;
* blocks force pushes and deletion of `main`;
* has **no bypass list**: the owner goes through the same path. In an emergency, the ruleset can be
  disabled temporarily, which is a visible, deliberate action.

Rulesets and branch protection are not available for private repositories on the GitHub Free plan, so this
decision requires the repository to be public, as already decided in ADR-0005.

### Consequences

* Good, because `main` only receives changes that passed CI, so Argo CD never applies an unchecked change
* Good, because commit message validation in CI is a single check on the pull request title and body
* Good, because branch commits can be messy work in progress without polluting `main`
* Good, because pull requests are the natural place for future automation (plans, dependency updates,
  release pull requests)
* Good, because the changelog is generated from the squashed commits on `main`, one entry per pull request
* Bad, because even a one-line fix needs a branch and a pull request; `gh pr create --fill` and auto-merge
  keep this small
* Bad, because a pull request that mixes two logical changes becomes a single commit; discipline is still
  needed when opening pull requests
* Bad, because the `BREAKING CHANGE:` footer must be written in the pull request body, which is easy to
  forget; CI must check it
* Neutral, because with no bypass, an emergency fix waits for CI unless the ruleset is disabled

### Confirmation

* A direct `git push` to `main` is rejected by GitHub
* A pull request with a non-conforming title, or with `!` but no `BREAKING CHANGE:` footer in the body,
  cannot be merged
* The repository settings show only squash merge enabled, with `PR_TITLE` / `PR_BODY` as the squash
  commit format
* Every commit on `main` after this ADR is a squashed pull request with a Conventional Commits subject

## Pros and Cons of the Options

### Direct pushes to `main`

CI runs after the push and only reports.

* Good, because there is no friction: commit and push
* Bad, because a broken change is already on `main`, and applied by Argo CD, when CI reports it
* Bad, because there is no place to review a plan before it is applied
* Bad, because bots that open pull requests (e.g. dependency updates) would still need a pull request
  flow, so there would be two workflows

### Pull requests with squash merge

Each pull request becomes one commit on `main`, with the pull request title and body as its message.

* Good, because CI gates `main` and validates a single message per change
* Good, because branch history does not need to be cleaned before merging
* Good, because "one pull request = one logical change" is easy to remember
* Bad, because granularity inside a pull request is lost
* Bad, because the breaking change footer lives in the pull request body

### Pull requests with rebase merge

Each commit of the branch lands on `main` individually, with linear history.

* Good, because atomic commits inside a pull request are preserved
* Bad, because CI must validate every commit of the pull request
* Bad, because the branch must be cleaned (`git rebase -i`, `fixup!`) before every merge

### Pull requests with merge commits

The GitHub default.

* Good, because the branch history is preserved exactly
* Bad, because "Merge pull request #N" commits do not follow Conventional Commits and clutter the history
  and the changelog
* Bad, because history is not linear

## More Information

* GitHub merge methods: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github>
* Squash commit message settings (`squash_merge_commit_title`, `squash_merge_commit_message`):
  <https://docs.github.com/en/rest/repos/repos#update-a-repository>
* Rulesets: <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets>
* This decision answers the contribution workflow follow-up of ADR-0003. The tools that implement the
  checks (hook manager, commit message linter, changelog generator) are decided in separate ADRs.
