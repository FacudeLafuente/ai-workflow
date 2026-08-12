# Skill overlay: spec-intake — BE profile

> Overlay applied on top of [`spec-intake.md`](spec-intake.md) when the active
> profile is `be`. Adds BE-specific inputs, DoR checks, and rubric rows.

## Additional inputs (BE)

- **HTTP contract** — for every endpoint the change adds or modifies:
  - Method + path (path style declared in `AGENTS.md`: kebab-case,
    snake_case, camelCase).
  - Request DTO / model shape.
  - Response DTO / model shape.
  - Documented status codes (2xx and every 4xx/5xx that maps to a known
    error).
  - Uniform error envelope mapping — which exception / error type triggers
    each documented status code.
- **Data touched:**
  - Entities / models the change reads or writes.
  - Tables / collections behind them.
  - Schema change (yes / no). If yes, migration strategy per `AGENTS.md`
    (EF migrations, SQL script, dbt, Alembic, Liquibase, …).
- **External integrations & events:**
  - Upstream services (Vault, KMS, external APIs).
  - Messages / events published or consumed.
  - Background jobs / schedulers.
- **Auth policy / guard** — the declared policy / filter / decorator name
  from `AGENTS.md`. Every non-public endpoint must name one.
- **Idempotency / concurrency** — idempotency-key rules, optimistic-locking
  strategy, retry semantics.
- **Feature flag / configuration** — key name and default value per env.

## Additional steps (BE)

- **Contract sanity-check.** For each declared endpoint:
  - Path follows the declared style (typical: kebab-case under `/api/…`).
  - Method matches semantics (GET pure, POST creates/triggers, PATCH partial,
    PUT full-replace idempotent, DELETE idempotent).
  - Auth policy / guard is declared or the endpoint is justified as public.
  - Every documented status code maps to a known exception / error type
    (per `AGENTS.md`).
  - Response type metadata (`[ProducesResponseType]` in .NET, `@ApiResponse`
    in Spring, `openapi()` in Nest, `responses:` in FastAPI, …) is declared
    for every status code.
- **Migrations check.** If the change requires a schema change, cross-check
  against the migrations flow declared in `AGENTS.md`. If no flow is
  declared, treat the change as `RED` in Phase 1 and escalate.
- **Mode B validation.** Cross-check the ticket against
  `templates/ticket.be.md` — headings must be `ToDo / AC / Contracts & Specs /
  Notes & Obvs`, in order.

## Additional DoR checklist rows (BE)

- [ ] HTTP contract complete per endpoint (method, path, request DTO,
      response DTO, status codes, error envelope mapping).
- [ ] Persistence impact declared (entities, tables, schema change y/n).
      If schema change: migrations flow confirmed against `AGENTS.md`; else
      escalate.
- [ ] Auth policy / guard declared or public-endpoint justification recorded.
- [ ] Uniform error envelope: every declared status code is mapped to a
      known exception / error type.
- [ ] External integrations / events declared (or `N/A`).
- [ ] Feature flag key + default per env declared (or `N/A`).
- [ ] Idempotency, pagination, concurrency, PII rules declared (or `N/A`).

## Additional rubric rows (BE)

Merge into the base rubric of `spec-intake.md`:

| # | Check                     | Pass criterion                                                                                                          |
|---|---------------------------|-------------------------------------------------------------------------------------------------------------------------|
| B1 | Contract completeness    | Every endpoint declares route, method, request DTO, response DTO, status codes, error envelope mapping.                 |
| B2 | Auth explicitness         | Every endpoint has a declared policy / guard or a public-endpoint justification recorded.                               |
| B3 | Persistence explicitness  | Entities/tables listed; schema-change flag set; if schema change, migrations flow decision is recorded or escalated.    |
| B4 | Error envelope mapping    | Every documented status code has a known exception / error type that triggers it (per `AGENTS.md`).                     |
| B5 | Response-type metadata    | Every status code is (or will be) declared via the repo's response-type metadata mechanism.                             |
| B6 | Integrations declared     | External services / events / jobs are declared (or explicitly `N/A`).                                                    |

## Readiness Report additions (BE)

Add these sections to the Readiness Report template from the base skill:

```md
## Contract

- `<METHOD>` `<path>` — policy `<name>` — 200 / 4xx …
- Request DTO: `<path/to/RequestDto>`
- Response DTO: `<path/to/ResponseDto>`
- Errors follow `<GlobalExceptionHandler / equivalent>`.

## Persistence

- Entities: `<list>`.
- Tables: `<list>`.
- Schema change: yes/no → strategy: `<name>` (per `AGENTS.md`).

## External integrations / events

- <service / event / job> — <role> **or** `N/A`.

## Non-functional (BE)

- Auth policy / guard: `<name>` (or `public — <reason>`).
- Idempotency: <yes/no + strategy>.
- Pagination / sorting / filtering: <describe> or `N/A`.
- PII / exposure: <describe>.
- Feature flag: <key> (default per env).
```

## Tools / MCP hooks (BE)

- **OpenAPI MCP** (optional / roadmap) — diff the proposed contract against
  the live Swagger / OpenAPI document (`/swagger/v1/swagger.json` or similar)
  and reject drift.
- **Context7 MCP** (if declared in `AGENTS.md`) — pull framework / library
  docs when the change touches unfamiliar APIs.
