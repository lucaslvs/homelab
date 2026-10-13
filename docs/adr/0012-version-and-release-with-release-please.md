---
status: accepted
date: 2026-10-13
decision-makers: Lucas Varlesse
---

# Version and release with release-please

## Context and Problem Statement

[ADR-0003](0003-use-conventional-commits.md) requires a `CHANGELOG.md` generated from the commit history
and left open which generator to use and whether to tag versions or milestones. It also defines a breaking
change in operational terms: a change that requires manual steps outside the automation, or risks losing
data or state. [ADR-0007](0007-contribute-through-pull-requests-with-squash-merge.md) made every commit on
`main` a squashed pull request, and protects `main` with a ruleset that has no bypass, so no tool can push
to `main` directly.

The homelab has no consumers in the sense a library has, but it is operated over time and its history is
part of the portfolio. Does it have versions, which format does the changelog follow, and which tool
produces both?

## Decision Drivers

* `CHANGELOG.md` is generated, never written by hand (ADR-0003)
* Work with the `main` ruleset: changelog and version changes must arrive through a pull request
  (ADR-0007)
* A version number that says something useful to whoever operates the homelab
* The owner decides *when* a release happens
* A changelog format recognized by the industry, telling the story of the project for portfolio readers
* As little custom workflow code as possible

## Considered Options

Versioning scheme:

* Semantic Versioning (SemVer)
* Calendar Versioning (CalVer)
* Manual milestone tags
* No versions

Tool:

* release-please
* git-cliff
* semantic-release
* cocogitto

## Decision Outcome

Chosen options: "Semantic Versioning (SemVer)", starting at `0.x`, and "release-please", because SemVer
computed from Conventional Commits gives the version number a meaning tied to ADR-0003's definition of a
breaking change, and release-please is the only tool that delivers the changelog, the version and the tag
through a pull request, so it works under the ADR-0007 ruleset without custom workflow code, and lets the
owner choose when to release by merging that pull request.

The decision is applied as follows:

**Versioning**

* Versions follow SemVer and are tagged `vX.Y.Z`, with a GitHub Release per version.
* While the version is `0.x` (`bump-minor-pre-major`), a breaking change bumps the minor version, like a
  `feat`; a `fix` bumps the patch version. In `0.x`, the number alone does not signal a breaking change; the
  "⚠ BREAKING CHANGES" section of the release does.
* `1.0.0` is declared by hand, with a `Release-As: 1.0.0` footer, when the homelab works from bare metal to
  applications entirely through the automation. From then on, a breaking change bumps the major version,
  which then means "this release requires manual steps or risks data; read the notes before applying".

**Release flow**

* release-please keeps a release pull request open that accumulates the changelog and the next version as
  pull requests are merged. Merging it updates `CHANGELOG.md`, creates the tag and the GitHub Release.
* The release pull request is opened with a **GitHub App** token, not the default `GITHUB_TOKEN`, because
  events created with `GITHUB_TOKEN` do not trigger workflows, so the required checks would never run and
  the pull request could never be merged under the ruleset.
* Its title follows the commit rules (e.g. `chore: release 0.5.0`), configured with
  `pull-request-title-pattern`, because the default `chore(main): …` uses a scope not allowed by
  [ADR-0006](0006-define-a-closed-list-of-commit-scopes.md).
* Release-please configuration files (`release-please-config.json`, `.release-please-manifest.json`) and
  any version file required by its release type live at the root, as files required by the tool (see
  [ADR-0005](0005-organize-the-repository-by-lifecycle-stage.md)).

**Changelog format**

* `CHANGELOG.md` uses the Conventional Changelog format produced by release-please: sections by commit
  type, the scope in bold at the start of each entry, links to the pull request and the commit, and a
  "⚠ BREAKING CHANGES" section at the top built from the `BREAKING CHANGE:` footers.
* **All** types of ADR-0003 are visible, with the default section titles, including `docs` (ADRs) and `ci`,
  because the whole evolution of the homelab is part of the story.
* The GitHub Release of a milestone (e.g. "host ready") may get a hand-written **Highlights** text on top
  of the generated notes. `CHANGELOG.md` itself is never edited by hand.

### Consequences

* Good, because the changelog, the version and the tag are produced with no manual writing
* Good, because everything reaches `main` through a pull request that passes the required checks
* Good, because the owner decides when to release, by merging the release pull request
* Good, because `BREAKING CHANGE:` footers become the upgrade notes at the top of each release
* Good, because the format is the standard of the Conventional Commits ecosystem
* Bad, because a GitHub App must be created and its credentials stored as GitHub Actions secrets
* Bad, because three or four files are added at the root
* Bad, because in `0.x` the version number does not distinguish a breaking change from a feature
* Bad, because the changelog is grouped by type, not by stage; the scope prefix carries the stage
* Neutral, because the changelog is only as good as the pull request titles

### Confirmation

* Merging a pull request into `main` opens or updates the release pull request, and its required checks
  run
* Merging the release pull request updates `CHANGELOG.md`, creates the `vX.Y.Z` tag and a GitHub Release
* No workflow pushes commits to `main` directly
* A breaking change in `0.x` bumps the minor version, and its footer appears under "⚠ BREAKING CHANGES"

## Pros and Cons of the Options

### Semantic Versioning (SemVer)

* Good, because the version is computed from the commit types, with no decision per release
* Good, because, after `1.0.0`, a major version signals a breaking change as defined in ADR-0003
* Bad, because version numbers can look unusual for a homelab

### Calendar Versioning (CalVer)

E.g. `2026.10.0`.

* Good, because it says when a release happened, natural for something always in production
* Bad, because a breaking change is not visible in the number

### Manual milestone tags

E.g. `foundations`, `host-ready`, `cluster-ready`.

* Good, because the tags tell the story of the project
* Bad, because releases depend on deciding and tagging by hand
* Neutral, because milestones can still be expressed as Highlights of SemVer releases

### No versions

* Good, because it is the simplest
* Bad, because there is no way to name the state of the homelab at a point in time other than a commit hash

### release-please

GitHub Action, by Google.

* Good, because the changelog, version and tag arrive through a release pull request, compatible with the
  ruleset
* Good, because it is built for squash merges and Conventional Commits
* Good, because the owner chooses when to release
* Bad, because the release pull request needs a GitHub App token (or a personal access token) to trigger
  the required checks
* Bad, because its configuration files add to the root

### git-cliff

A Rust binary that generates changelogs and computes the next version.

* Good, because it has the most flexible changelog templates (e.g. grouping by stage)
* Good, because it runs locally and in CI, installed through mise
* Bad, because it does not tag, commit or open pull requests: the release flow must be written around it

### semantic-release

Node.js; releases on every merge to `main`.

* Good, because releases are fully automatic
* Bad, because committing `CHANGELOG.md` requires pushing to `main` directly, which the ruleset forbids;
  without that, the changelog lives only in GitHub Releases, contrary to ADR-0003
* Bad, because every merge becomes a release, without the owner's choice

### cocogitto

`cog bump`.

* Good, because the same tool could validate commits, generate the changelog and bump versions
* Bad, because it creates the version commit and tag locally and expects a direct push to `main`

## More Information

* Semantic Versioning: <https://semver.org/>
* Calendar Versioning: <https://calver.org/>
* Keep a Changelog (considered as a format, not adopted because it is meant to be hand-written and has
  sections without a Conventional Commits type): <https://keepachangelog.com/>
* release-please: <https://github.com/googleapis/release-please>
* release-please action (GitHub App or personal access token for the release pull request):
  <https://github.com/googleapis/release-please-action>
* Creating a GitHub App token in workflows: <https://github.com/actions/create-github-app-token>
* git-cliff: <https://git-cliff.org/>
