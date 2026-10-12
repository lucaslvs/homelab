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

## Adding a new ADR

1. Copy `adr-template.md` to `NNNN-title-in-kebab-case.md`, using the next free number.
2. Fill in context, drivers and options; leave `status: proposed`.
3. Once the decision is made, set `status: accepted` and update the index above.
4. Never delete or renumber an ADR. To change a decision, write a new ADR and mark the old one
   `superseded by ADR-NNNN`.
