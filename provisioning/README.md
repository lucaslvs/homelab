# provisioning

Creates the Incus resources on the host declaratively: networks, profiles and
[guests](../docs/glossary.md#guest). **Current tool:** OpenTofu with the Incus provider.

> **Status:** not implemented yet.

## Layout

```
provisioning/
├── networks/             # root module, own state
├── profiles/             # root module, own state
└── guests/               # root module, own state
```

* One root module per subject, each with its own state. Root modules reference each other's resources by
  name (passed as variables), never by reading each other's state.
* Apply order: `networks → profiles → guests`; destroy in reverse.
* A `modules/` directory for shared child modules is added only once real repetition appears.
* State is never committed. The state backend is a pending decision.

See [ADR-0005](../docs/adr/0005-organize-the-repository-by-lifecycle-stage.md#provisioning).
