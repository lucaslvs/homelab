---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Record architecture decisions using MADR

## Context and Problem Statement

This homelab is being built from scratch and doubles as a public technical portfolio. Many significant
choices lie ahead (virtualization model for Kubernetes nodes, cluster datastore, repository layout, host
bootstrap strategy), and several of them will be revisited as the project matures. Drafts and options are
often prepared with the help of an AI assistant, while the decisions themselves are made by the owner.

How do we keep a durable, reviewable record of *what* was decided, *why*, and *which alternatives were
discarded*, so the reasoning survives beyond chat sessions and commit messages?

## Decision Drivers

* Capture the rationale ("why"), not only the outcome
* Make discarded alternatives and their trade-offs visible to readers of the portfolio
* Make explicit who decided, separating drafted proposals from accepted decisions
* Allow decisions to evolve (superseded, deprecated) without rewriting history
* Plain Markdown, readable directly on GitHub, no extra tooling required

## Considered Options

* MADR (full template)
* MADR (minimal template)
* Michael Nygard's ADR format
* No ADRs (rely on commit messages, README and ARCHITECTURE.md)

## Decision Outcome

Chosen option: "MADR (full template)", because its explicit "Considered Options" and "Pros and Cons"
sections match the working model of this project (options are drafted, the owner chooses), and its YAML
front matter makes status and decision-makers explicit.

### Consequences

* Good, because every decision documents the alternatives that were weighed, not just the winner
* Good, because the `status` field distinguishes `proposed` drafts from `accepted` decisions
* Good, because front matter is machine-readable, enabling indexes or rendered sites later
* Bad, because ADRs are longer to write; optional sections may be removed when they add nothing
* Bad, because the extra friction may tempt skipping ADRs for small decisions

### Confirmation

* Every significant architectural or tooling decision lands as a numbered ADR in `docs/adr/`, created from
  `adr-template.md`, in the same change that implements it (or before it)
* An ADR is only `accepted` after the owner reviews it; drafts stay `proposed`
* ADRs are never deleted: superseded decisions get `status: superseded by ADR-NNNN` and the new ADR links
  back to the old one

## Pros and Cons of the Options

### MADR (full template)

* Good, because options, drivers and trade-offs each have a dedicated section
* Good, because it is actively maintained and widely recognized
* Neutral, because most sections are optional, so it can shrink when needed
* Bad, because it is the most verbose option

### MADR (minimal template)

* Good, because it keeps the structured options list with less overhead
* Bad, because pros and cons of discarded options are not recorded

### Michael Nygard's ADR format

* Good, because it is the original, short and fast to write
* Good, because tooling such as `adr-tools` supports it
* Bad, because alternatives are buried in free-form context text
* Bad, because there is no structured metadata (decision-makers, dates)

### No ADRs

* Good, because there is zero overhead
* Bad, because rationale is lost or scattered across commits and chat sessions
* Bad, because the portfolio shows outcomes but not engineering judgment

## More Information

* MADR: <https://adr.github.io/madr/>
* Michael Nygard, "Documenting Architecture Decisions" (2011):
  <https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions>
* File naming: `NNNN-title-in-kebab-case.md`, numbered sequentially, never reused
