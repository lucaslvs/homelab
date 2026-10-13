# Architecture Decision Records

Significant decisions of this homelab are recorded here using [MADR](https://adr.github.io/madr/)
(see [ADR-0001](0001-record-architecture-decisions-using-madr.md)).

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](0001-record-architecture-decisions-using-madr.md) | Record architecture decisions using MADR | accepted |
| [0002](0002-use-english-as-repository-language.md) | Use English as the repository language | accepted |
| [0003](0003-use-conventional-commits.md) | Use Conventional Commits | accepted |
| [0004](0004-adopt-a-shared-vocabulary-for-machines-and-roles.md) | Adopt a shared vocabulary for machines and roles | accepted |
| [0005](0005-organize-the-repository-by-lifecycle-stage.md) | Organize the repository by lifecycle stage | accepted |
| [0006](0006-define-a-closed-list-of-commit-scopes.md) | Define a closed list of commit scopes | accepted |
| [0007](0007-contribute-through-pull-requests-with-squash-merge.md) | Contribute through pull requests with squash merge | accepted |
| [0008](0008-manage-workstation-tool-versions-with-mise.md) | Manage workstation tool versions with mise | accepted |
| [0009](0009-run-tasks-with-mise.md) | Run tasks with mise | accepted |
| [0010](0010-manage-git-hooks-with-lefthook.md) | Manage git hooks with lefthook | accepted |

## Adding a new ADR

1. Copy `adr-template.md` to `NNNN-title-in-kebab-case.md`, using the next free number.
2. Fill in context, drivers and options; leave `status: proposed`.
3. Once the decision is made, set `status: accepted` and update the index above.
4. Never delete or renumber an ADR. To change a decision, write a new ADR and mark the old one
   `superseded by ADR-NNNN`.
