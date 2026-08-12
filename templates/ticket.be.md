<!--
=============================================================================
 IMMUTABLE TICKET FORMAT — ASDD (Agentic Spec-Driven Development) — BE profile
=============================================================================
 Authoring rules (NOT part of the ticket description — remove before pasting
 into your ticketing system if it renders HTML comments):

 - Granular:       one vertical slice → one ticket → one PR.
 - Parallelizable: dependencies are explicit in `Notes & Obvs`
                   (`blocked by #ID` / `blocks #ID`).
 - Governable:     the ticket represents ONE stage of work — reviewable end-to-end.
 - Traceable:      every AC line maps to a Business/Product criterion.
 - Immutable:      the four headings below never change and never get reordered.
                   Empty section ⇒ `N/A`. Do not add extra top-level headings.
 - Format-exact:   do not rename `ToDo`, `AC`, `Contracts & Specs`, `Notes & Obvs`.
 - Self-contained: an implementer must be able to build the slice from this
                   ticket alone (no hidden context in chat/thread history).
 - Repo conventions: honor the rules declared in the consumer repo's AGENTS.md
                   (layering, auth policy, uniform error envelope, routing
                   style, DTO placement, ORM / persistence, testing harness).

 The four headings for the BE profile are:
   ## ToDo
   ## AC
   ## Contracts & Specs
   ## Notes & Obvs
=============================================================================
-->

# <TICKET-ID> — <Short imperative title>

## ToDo

<!--
Atomic, verifiable checklist items. Each item is something a reviewer can tick
off from the diff (a file created, a method added, an endpoint wired, a
mapping added, a policy applied, a test class added). No verbs like
"investigate" or "consider" — those belong in Phase 1, not in a ticket.
-->

- [ ] <atomic action 1>
- [ ] <atomic action 2>
- [ ] Auth annotation applied to the new/changed endpoint (per the repo's auth
      convention declared in AGENTS.md)
- [ ] Response-type / OpenAPI metadata declared for every documented status
      code
- [ ] DTO / model mapping added to the repo's canonical mapping layer
      (AutoMapper profile / MapStruct mapper / manual mapper — as declared in
      AGENTS.md)
- [ ] Tests added under the correct test project / package following the
      repo's testing harness
- [ ] Build / test / migration command list included in the PR description
      (Dev runs them)

## AC

<!--
Given / When / Then. Each line traces to a Business/Product criterion.
Always include an explicit "Edge cases / states covered" line so the loop
verifier can enforce full coverage.
-->

- **Given** <precondition> **When** <action> **Then** <observable outcome>.
- **Given** <precondition> **When** <action> **Then** <observable outcome>.
- **Edge cases / states covered:** empty result, not-found (404),
  unauthorized (401), forbidden (403), validation error (400 with `errors`
  payload), conflict (409), rate-limit (429), partial failure, idempotent
  retry, concurrent modification, upstream unavailable (503). Mark any that
  do **not** apply with `N/A — <reason>`; never silently omit.

## Contracts & Specs

<!--
Backend contract of the slice. This is the section a frontend / integrator can
read to consume the change without opening the diff.
-->

### HTTP endpoint(s)

| Method | Path                                | Auth policy             | Success | Documented errors    |
|--------|-------------------------------------|-------------------------|---------|----------------------|
| `POST` | `/api/<resource>/<sub-resource>`    | `<PolicyOrGuardName>`   | `200`   | `400`, `403`, `404`  |

<!--
Path style follows the repo's convention declared in AGENTS.md (kebab-case,
snake_case, camelCase). Auth policy follows the repo's naming (policies,
guards, filters, decorators).
-->

### Request

```jsonc
// POST /api/<resource>/<sub-resource>
{
  "<field>": "<value>"
}
```

- Request DTO / model: `<path/to/Name>RequestDto.<ext>`
- Validation rules: <list, or `N/A`>.

### Success response

```jsonc
// 200 OK
{
  "<field>": "<value>"
}
```

- Response DTO / model: `<path/to/Name>ResponseDto.<ext>`
- Mapped from: `<domain type>` via `<mapping profile>`.

### Error responses (uniform envelope)

<!--
All errors follow the repo's declared error envelope (see AGENTS.md → error
handling). Example below; adapt to your envelope shape.
-->

```jsonc
{
  "message": "<ERROR_CODE_OR_TEXT>",
  "statusCode": 400,
  "isHandled": true,
  "errors": null
}
```

| Status | Exception / error type thrown          | Trigger                                    |
|--------|----------------------------------------|--------------------------------------------|
| 400    | `ValidationException("<CODE>")`        | <precondition>                             |
| 403    | Policy failure / `ForbiddenException`  | Caller lacks required policy               |
| 404    | `NotFoundException("<CODE>")`          | Resource does not exist                    |
| 409    | `<...Conflict>`                        | `N/A` unless the ticket declares conflict  |
| 503    | `<UpstreamUnavailableException>`       | External dependency down                   |

### Persistence / schema impact

- Entities touched: `<path/to/Entity>.<ext>`.
- Tables touched: `<table>` via `<DbContext / repository>`.
- Schema change: `N/A` **or** describe the change and the migration/script
  strategy (cross-check with the migrations flow declared in AGENTS.md).

### External integrations / events

- `<external service / event / message>` **or** `N/A`.

### Feature flag / configuration

- Key: `<key>` (or `N/A`), default value per environment.

### Non-functional

- Idempotency: <yes/no + strategy> or `N/A`.
- Pagination / sorting / filtering: <describe> or `N/A`.
- Performance target: <p95 ms / max payload / max rows> or `N/A`.
- PII / data exposure: <describe> or `N/A`.
- Rate-limit: <describe> or `N/A`.

## Notes & Obvs

<!--
Everything a reviewer needs that is not part of the contract: touchpoints,
dependencies, governance/risk, out-of-scope, manual commands.
-->

- **Repo touchpoints:**
  - `<Controller path>`
  - `<Service interface path>`
  - `<Service implementation path>`
  - `<Repository path>`
  - `<Mapping path>`
  - `<Test path>`
- **Dependencies:** `blocked by #<ID>` / `blocks #<ID>` (or `none`).
- **Governance / risk:**
  - Auth: `<policy/guard name>` (must not be relaxed).
  - Data exposure: <describe> or `N/A`.
  - Migration / breaking change: <describe> or `N/A`.
  - Backward compatibility: <describe> or `N/A`.
- **Out of scope:** <list explicitly>.
- **Manual commands the Dev will run (AI never runs these):**
  - `<restore command>` (e.g. `dotnet restore`, `npm install`, `mvn install`)
  - `<build command>` (e.g. `dotnet build -c Release`, `mvn compile`)
  - `<test command>` (e.g. `dotnet test`, `mvn test`, `pytest`)
  - `<migration command>` (only if the ticket has schema impact and the
    migrations flow is confirmed)
