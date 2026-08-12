# Skill overlay: ticket-author — BE profile

> Overlay applied on top of [`ticket-author.md`](ticket-author.md) when the
> active profile is `be`. Uses `templates/ticket.be.md`.

## Heading order (immutable, BE)

```text
## ToDo
## AC
## Contracts & Specs
## Notes & Obvs
```

## ToDo discipline (BE)

Whenever applicable to the slice, add these atomic items to the `ToDo`:

- [ ] Auth annotation applied to the new/changed endpoint (per `AGENTS.md`:
      `[Authorize(Policy = PolicyNames.<X>)]` in .NET, `@PreAuthorize` in
      Spring, `@UseGuards(<Guard>)` in Nest, or equivalent).
- [ ] Response-type metadata declared for every documented status code
      (`[ProducesResponseType]`, `@ApiResponse`, `responses:` in FastAPI).
- [ ] DTO mapping added to the declared mapping layer (AutoMapper profile,
      MapStruct mapper, manual mapper) for any new DTO.
- [ ] Repository method added to the declared repositories layer, exposed via
      an interface.
- [ ] Service method added to the declared services layer, throwing typed
      exceptions that map to the uniform error envelope.
- [ ] DTOs placed in the declared layers (Web-layer vs Domain-layer).
- [ ] Migration authored and applied (only if the slice has schema impact and
      the migrations flow is confirmed).
- [ ] Tests added under the correct test project / package; integration tests
      cover every documented status code path.
- [ ] Build / test / migration command list included in the PR description
      (Dev runs them).

## Contracts & Specs discipline

Fill the third section per this shape (use `N/A` for any inapplicable
subsection):

```md
## Contracts & Specs

### HTTP endpoint(s)

| Method | Path                                | Auth policy / guard | Success | Documented errors    |
|--------|-------------------------------------|---------------------|---------|----------------------|
| `POST` | `/api/<resource>/<sub-resource>`    | `<name>`            | `200`   | `400`, `403`, `404`  |

### Request

```jsonc
// POST /api/<resource>/<sub-resource>
{ "<field>": "<value>" }
```

- Request DTO: `<path/to/Name>RequestDto.<ext>`.
- Validation rules: <list, or `N/A`>.

### Success response

```jsonc
// 200 OK
{ "<field>": "<value>" }
```

- Response DTO: `<path/to/Name>ResponseDto.<ext>`.
- Mapped from: `<domain type>` via `<mapping profile>`.

### Error responses (uniform envelope)

<Envelope shape per `AGENTS.md`. Example:>

```jsonc
{ "message": "<CODE>", "statusCode": 400, "isHandled": true, "errors": null }
```

| Status | Exception / error type                 | Trigger                                    |
|--------|----------------------------------------|--------------------------------------------|
| 400    | `ValidationException("<CODE>")`        | <precondition>                             |
| 403    | Policy failure / `ForbiddenException`  | Caller lacks required policy               |
| 404    | `NotFoundException("<CODE>")`          | Resource does not exist                    |
| 409    | `<...Conflict>`                        | <precondition>                             |
| 503    | `<UpstreamUnavailable>`                | External dependency down                   |

### Persistence / schema impact

- Entities touched: `<list>`.
- Tables touched: `<list>` via `<DbContext / equivalent>`.
- Schema change: `N/A` **or** describe migration / script strategy.

### External integrations / events

- <service / event / job> or `N/A`.

### Feature flag / configuration

- Key: `<key>` (or `N/A`), default value per environment.

### Non-functional

- Idempotency: <yes/no + strategy> or `N/A`.
- Pagination / sorting / filtering: <describe> or `N/A`.
- Performance target: <p95 ms / max payload / max rows> or `N/A`.
- PII / data exposure: <describe> or `N/A`.
- Rate-limit: <describe> or `N/A`.
```

## Notes & Obvs discipline (BE)

Include these blocks when applicable:

- **Repo touchpoints** — one file per line (controller, service interface,
  service implementation, repository, mapping, tests, migration, dbcontext).
- **Dependencies** — `blocked by #<ID>` / `blocks #<ID>`.
- **Governance / risk:**
  - Auth: policy name (must not be relaxed).
  - Data exposure: describe or `N/A`.
  - Migration / breaking change: describe or `N/A`.
  - Backward compatibility: describe or `N/A`.
- **Out of scope** — explicit list.
- **Manual commands** — restore / build / test / migration commands — copied
  from `AGENTS.md`.

## Additional rubric rows (BE)

Merge into the base rubric of `ticket-author.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Endpoint table               | Method + path + policy + status codes declared for every endpoint.                                          |
| B2 | Request/response DTOs        | Paths + shapes declared; validation rules stated (or `N/A`).                                                |
| B3 | Error envelope table         | Every status code has an exception/error mapping.                                                           |
| B4 | Persistence explicitness     | Entities + tables listed; schema change y/n; migration strategy referenced if yes.                          |
| B5 | Response-type metadata       | ToDo explicitly requires `[ProducesResponseType]` / `@ApiResponse` for every status code.                   |
| B6 | Mapping / DTO placement      | Mapping profile updated; DTOs placed in the declared layers.                                                |

## Tools / MCP hooks (BE)

- Ticket template: `templates/ticket.be.md`.
- OpenAPI MCP (roadmap) — diff proposed contract vs live Swagger.
- Ticketing MCP (roadmap) — create/link Work Items directly.
