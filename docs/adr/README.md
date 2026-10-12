# Architecture Decision Records

Significant decisions of this homelab are recorded here using [MADR](https://adr.github.io/madr/)
(see [ADR-0001](0001-record-architecture-decisions-using-madr.md)).

## Index

| ADR | Title | Status |
|-----|-------|--------|
| [0001](0001-record-architecture-decisions-using-madr.md) | Record architecture decisions using MADR | accepted |

## Adding a new ADR

1. Copy `adr-template.md` to `NNNN-title-in-kebab-case.md`, using the next free number.
2. Fill in context, drivers and options; leave `status: proposed`.
3. Once the decision is made, set `status: accepted` and update the index above.
4. Never delete or renumber an ADR. To change a decision, write a new ADR and mark the old one
   `superseded by ADR-NNNN`.
