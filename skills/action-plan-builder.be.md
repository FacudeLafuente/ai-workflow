# Skill overlay: action-plan-builder — BE profile

> Overlay applied on top of [`action-plan-builder.md`](action-plan-builder.md)
> when the active profile is `be`.

## Verticality for BE

A vertical BE slice covers, end-to-end:

- Entity / DbContext change (if schema impact).
- Migration (if schema impact).
- Repository method(s).
- Service method(s) + interface update.
- DTOs (Web + Domain).
- Mapping (AutoMapper profile / MapStruct / manual mapper).
- Controller / handler + auth annotation + response-type metadata.
- Tests (unit + integration where applicable).

Avoid these horizontal cuts:

- "All entities first, then all repositories, then all services" — nothing
  to ship after each step.
- "One controller / no service" — controllers with logic bypass the layering.
- "Migration alone, no consumer" — schema change without any service using
  it produces a slice with no observable behavior.

## Typical split axes for oversized BE slices

If a slice exceeds ~300 diff lines or two endpoints, split along:

- **Read vs write** — one slice for read endpoints, another for write
  endpoints.
- **Schema-migration slice** vs **service slice** vs **controller slice** —
  only when the migration is large enough to justify its own PR and can ship
  without breaking anything (behind a flag or unused column).
- **Endpoint boundaries** — one slice per endpoint when each endpoint carries
  substantive service logic.
- **Feature-flag phases** — introduce behavior behind a flag; migrate
  callers; remove flag — three slices.

## Cross-cutting slices that must ship first (BE)

- **New exception → status code mapping** in the global error handler.
- **New auth policy / guard** in the policy names / guard registry.
- **New shared DTO / envelope shape** consumed by multiple endpoints.
- **New migration** that later slices depend on (only if the migrations flow
  is confirmed).
- **New mapping profile** consumed by multiple endpoints.

## Additional rubric rows (BE)

Merge into the base rubric of `action-plan-builder.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Vertical BE slice            | Each slice covers entity → repo → service → DTO → mapper → controller → tests (or explains why not).        |
| B2 | Layering explicit            | Slice describes each layer it touches; no controller-with-DAL leakage proposed.                             |
| B3 | Cross-cutting order          | Error-envelope mappings, auth policies, mapping profiles, migrations ship before their consumers.           |
| B4 | Migrations gating            | Any slice with schema impact is either ordered after a confirmed migrations decision or explicitly blocked. |
| B5 | Contract discipline per slice | Each slice names route, method, auth, DTOs, status codes it introduces or modifies.                        |

## Action Plan additions (BE)

Add a "Contracts summary" block after the slice table:

```md
## Contracts summary

| Slice | Endpoint(s)                              | Auth       | Status codes           | Schema change |
|-------|------------------------------------------|------------|------------------------|---------------|
| S1    | `POST /api/<...>`                        | `<policy>` | 200, 400, 403, 404     | no            |
| S2    | `GET /api/<...>`                         | `<policy>` | 200, 403               | no            |
| S3    | migration + `POST /api/<...>`            | `<policy>` | 201, 400, 409          | yes           |
```

And a "Cross-cutting order" block:

```md
## Cross-cutting order

1. S0 — new error mapping in the global handler.
2. S1 — new auth policy.
3. S2 — first consumer slice.
```

## Tools / MCP hooks (BE)

- Repository read.
- Mermaid rendering for the dependency graph.
- Ticketing MCP (roadmap).
