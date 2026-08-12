# Skill overlay: implementation-loop — BE profile

> Overlay applied on top of [`implementation-loop.md`](implementation-loop.md)
> when the active profile is `be`.

## Implementation order (BE)

Implement the slice top-down through the layers declared by `AGENTS.md`.
Typical order (skip a layer if the slice does not touch it):

1. **Entity / DbContext** (if schema impact) — only if the migrations flow is
   confirmed for this ticket; otherwise stop and escalate.
2. **Migration** — authored per the declared migrations mechanism (EF Core
   `dotnet ef migrations add`, JPA / Liquibase changelog, Alembic revision,
   dbt model, etc.). Do **not** run the migration — list the exact command
   for the Developer.
3. **Repository** — under the declared repositories layer, implementing an
   interface declared in the abstractions layer.
4. **Service** — under the declared services layer, implementing an interface
   declared in the abstractions layer. Throw the declared typed exceptions so
   the uniform error envelope maps correctly. **Do not** catch-and-swallow.
   **Do not** add ad-hoc error shapes.
5. **DTOs:**
   - Web-facing DTOs in the declared Web-layer folder (Requests / Responses).
   - Domain / application DTOs in the declared application layer.
   - Never leak an entity through the HTTP boundary.
6. **Mapper** — add the mapping entry to the declared mapping layer
   (AutoMapper profile / MapStruct mapper / manual mapper).
7. **Controller / handler:**
   - Use the declared attributes / decorators (routing, content-type,
     versioning).
   - Apply the declared auth annotation on every non-public endpoint.
   - Declare response-type metadata for **every** status code named in the
     ticket.
   - Delegate to the service. No business logic in the controller.
8. **Tests** — under the declared test project / package. Cover every AC
   line and every non-`N/A` state from the Phase 0 matrix.
9. **OpenAPI / Swagger** — ensure the generated document reflects the new
   endpoint. If the repo uses annotations, they must be complete.

## Discipline (BE)

- **Layering** — Controller → Service → Repository. No DB calls from
  controllers. No business logic in controllers.
- **Uniform error envelope** — errors flow through the declared global
  handler; no ad-hoc `return BadRequest(...)` with bespoke shapes.
- **Auth** — the declared auth annotation is present on every non-public
  endpoint. No relaxation of an existing policy.
- **Contract discipline** — routing style respected, response-type metadata
  matches every documented code, DTO placement correct, mapping wired.
- **Idempotency** — declared strategy applied when the ticket requires it.
- **No commands** — do not invoke build/test/migration/`git`. List them.

## Additional rubric rows (BE)

Merge into the base rubric of `implementation-loop.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Layering                     | Controller → Service → Repository respected. No business logic in controllers. No DAL calls from controllers. |
| B2 | Uniform error envelope       | Errors flow through the declared global handler; no ad-hoc error shapes.                                    |
| B3 | Auth annotation              | Declared auth annotation present on every non-public endpoint.                                              |
| B4 | Response-type metadata       | Every documented status code has metadata declared.                                                         |
| B5 | Routing style                | Route style respected (typical: kebab-case under `/api/…`).                                                 |
| B6 | DTO placement                | Web-layer DTOs and Domain-layer DTOs in the declared folders; no cross-layer leaks.                         |
| B7 | Mapping wired                | New DTO ↔ Domain mapping added to the declared mapping profile.                                             |
| B8 | Migrations discipline        | Schema change authored per the declared flow; not silently applied; not skipped.                            |

## Implementation Handoff Report additions (BE)

Add a **Contract delta** block:

```md
## Contract delta

- `<METHOD> <path>` — policy `<name>` — 200 / 4xx …
- Request DTO: `<path>`
- Response DTO: `<path>`
- Errors follow `<global handler path>`.
- Response-type metadata: declared for 200, 400, 403, 404, …
- OpenAPI: `<endpoint>` will appear in the generated document.
```

And a **Migration delta** block when applicable:

```md
## Migration delta

- Migration name: `<Name>`.
- Command the Dev will run: `<exact migration command>`.
- Rollback strategy: <describe>.
```

## Tools / MCP hooks (BE)

- Repo read/write (edit files under Web / Application / Domain / Persistence
  layers + Tests).
- Subagent for the reviewer step.
- OpenAPI MCP (roadmap) — cross-check the generated Swagger against the
  ticket's declared contract.
- **No** command execution (`dotnet`, `mvn`, `gradle`, `npm`, `pip`, `git`).
