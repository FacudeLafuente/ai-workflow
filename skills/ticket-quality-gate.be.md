# Skill overlay: ticket-quality-gate — BE profile

> Overlay applied on top of [`ticket-quality-gate.md`](ticket-quality-gate.md)
> when the active profile is `be`.

## BE format check

The four immutable headings must be:

```text
## ToDo
## AC
## Contracts & Specs
## Notes & Obvs
```

Any renamed, reordered, or missing heading → reject.

## BE-specific repo-conventions checks

For each ticket, confirm `Contracts & Specs` explicitly names:

- Route (declared path style — typical: kebab-case under `/api/…`) + method.
- Auth policy / guard (per `AGENTS.md`) or public-endpoint justification.
- Uniform error envelope mapping (status ↔ exception ↔ trigger).
- DTO placement (Web vs Domain) and mapping profile entry.
- Response-type metadata (`[ProducesResponseType]` / `@ApiResponse` /
  `responses:`) for every declared status code.
- Persistence impact + schema-change strategy (or `N/A`).
- External integrations / events (or `N/A`).
- Non-functional (idempotency / pagination / PII / rate-limit) (or `N/A`).

And confirm `Notes & Obvs` explicitly names:

- Repo touchpoints (controller, service, repository, mapping, tests,
  migration, dbcontext).
- Dependencies (`blocked by` / `blocks`).
- Governance / risk (auth, data exposure, migration, backward compatibility).
- Manual commands (restore / build / test / migration).

## BE landmines to sweep

- **Schema change without a confirmed migrations flow** — reject.
- **Breaking API change** on a shipped route without a versioning strategy —
  reject.
- **Missing auth policy** without a public-endpoint justification — reject.
- **New exception → status code mapping** not documented in `Contracts &
  Specs` — reject / request changes.
- **DTO leak** across layers (returning an entity from a controller) — reject.
- **Ad-hoc error shape** in a controller instead of throwing the typed
  exception — reject.
- **Missing response-type metadata** for a declared status code — reject.

## Additional rubric rows (BE)

Merge into the base rubric of `ticket-quality-gate.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Endpoint table complete      | Method + path + policy + status codes declared for every endpoint.                                          |
| B2 | Error envelope table         | Every declared status code has an exception / error mapping.                                                |
| B3 | DTOs & mapping               | Request / response DTO paths + mapping profile entry named.                                                 |
| B4 | Response-type metadata       | ToDo requires `[ProducesResponseType]` / `@ApiResponse` for every status code.                              |
| B5 | Persistence / migrations     | Schema change flagged; migration strategy referenced (or `N/A`).                                            |
| B6 | Auth / policy explicit       | Policy / guard named; not relaxed.                                                                          |

## Governance Report additions (BE)

Add these columns to the report table from the base template:

| Endpoint tbl | Errors tbl | DTOs & mapping | Metadata | Auth | Migrations |
|--------------|------------|----------------|----------|------|------------|
| ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

## Tools / MCP hooks (BE)

- Read-only access to the ticket bodies.
- Diff against `templates/ticket.be.md`.
- OpenAPI MCP (roadmap) — cross-check declared contracts vs live Swagger.
