# Skill overlay: action-plan-builder — FE profile

> Overlay applied on top of [`action-plan-builder.md`](action-plan-builder.md)
> when the active profile is `fe`.

## Verticality for FE

A vertical FE slice covers, end-to-end:

- Route registration (or route update).
- View / page component.
- Child components composing it.
- Store slice (Pinia / Redux / NgRx / Zustand) if new state is needed.
- Repository / data-access hook (via the declared HTTP client abstraction).
- Types / models.
- Tests (component + store + repository as applicable, rendered via the
  declared harness).
- Design fidelity (Figma reference cited in the ticket).

Avoid these horizontal cuts (they produce un-shippable slices):

- "All types first, then all components, then all stores" — nothing to ship
  after each step.
- "One store action per slice" — creates orphan slices with no UI consumer.

## Typical split axes for oversized FE slices

If a slice exceeds ~300 diff lines or two screens, split along:

- **Read vs write** — one slice for the read/listing screen, another for the
  create/edit/delete flow.
- **Empty/error states** — one slice for the happy path, another for a
  substantial error-recovery flow (only when the error UX has its own AC lines).
- **Cross-module contract** — when a slice extracts a shared component into
  a design-system layer, that extraction becomes its own slice.
- **Feature-flag phases** — one slice behind flag `off` (implementation),
  one slice for flag `on` migration if applicable.

## Cross-cutting slices that must ship first (FE)

- **Design-system additions** — new token, new component variant, new
  primitive — must ship before any consumer slice.
- **Shared type / model changes** — updating a domain type consumed by
  multiple modules ships first.
- **HTTP client / auth changes** — updates to the HTTP client abstraction,
  token acquisition, error taxonomy — ship first.
- **Route / permission changes** — a new access level or route-meta key ships
  before slices that use it.

## Additional rubric rows (FE)

Merge into the base rubric of `action-plan-builder.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | Vertical FE slice            | Each slice covers route → view → component → store → data-access → tests (or explains why not).             |
| F2 | Design coverage per slice    | Each slice cites the Figma nodes and design-system components it implements; empty/error states listed.     |
| F3 | Feature-flag phasing         | If a flag is involved, the plan documents the flag's default state per env and the migration path.          |
| F4 | Cross-cutting order          | Design-system / shared-type / HTTP-client / route-meta slices ship before their consumers.                  |
| F5 | Consumed BE contracts        | Each slice that calls a BE endpoint either uses a shipped one or is `blocked by` a BE ticket.               |

## Action Plan additions (FE)

Add a "BE dependencies" block after the slice table:

```md
## BE dependencies

- Slice S<n> consumes `<METHOD> <path>` — status: SHIPPED / IN-FLIGHT.
  - If IN-FLIGHT: `blocked by #<BE-TICKET-ID>`.
```

And a "Design coverage" block:

```md
## Design coverage

| Slice | Screens (Figma nodes) | Design-system components | State variants covered |
|-------|-----------------------|--------------------------|------------------------|
| S1    | <name / node-id>      | <component list>         | empty, loading, error  |
| S2    | …                     | …                        | …                      |
```

## Tools / MCP hooks (FE)

- Repository read (for touchpoint validation).
- Figma MCP (roadmap).
- Ticketing MCP (roadmap).
