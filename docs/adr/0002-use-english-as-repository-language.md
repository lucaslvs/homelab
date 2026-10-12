---
status: accepted
date: 2026-10-12
decision-makers: Lucas Varlesse
---

# Use English as the repository language

## Context and Problem Statement

The repository is meant to be a public showcase of DevOps and infrastructure work. The owner's native
language is Brazilian Portuguese, and the initial design draft (`ARCHITECTURE.md`) was written in
Portuguese. Which language should be used for documentation, ADRs, code comments, file names and commit
messages?

## Decision Drivers

* Reach of the portfolio, including international recruiters and engineers
* Consistency with the ecosystem: tool documentation, error messages, upstream issues are in English
* Ease of writing and reviewing for the owner
* A single, consistent convention across the whole repository

## Considered Options

* English everywhere
* Portuguese everywhere
* Mixed: documentation and ADRs in Portuguese; code, file names and commits in English

## Decision Outcome

Chosen option: "English everywhere", because it maximizes the reach of the portfolio and keeps the
repository consistent with the language of the tools and communities it builds on.

### Consequences

* Good, because the repository is readable by the widest audience
* Good, because names, docs and upstream references share one language
* Bad, because writing documentation takes more effort than in the native language
* Neutral, because `ARCHITECTURE.md` is a transitional exploration draft in Portuguese; it is expected to be
  superseded by English ADRs and docs rather than translated

### Confirmation

* New documentation, ADRs, comments and commit messages are reviewed for English
* The transitional `ARCHITECTURE.md` draft is exempt until it is retired

## Pros and Cons of the Options

### English everywhere

* Good, because of the broadest audience and ecosystem consistency
* Bad, because of the higher writing effort for the owner

### Portuguese everywhere

* Good, because it is the most natural to write and review
* Bad, because it limits the audience of a public portfolio
* Bad, because it mixes awkwardly with English tool names, keys and identifiers

### Mixed

* Good, because docs are easy to write while code follows conventions
* Bad, because the language boundary is fuzzy (comments? READMEs inside tool folders?)
* Bad, because readers of the portfolio face two languages

## More Information

This decision covers repository content only. Conversations with collaborators or AI assistants may happen
in any language.
