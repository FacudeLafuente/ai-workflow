# Skill overlay: pr-handoff — BE profile

> Overlay applied on top of [`pr-handoff.md`](pr-handoff.md) when the active
> profile is `be`.

## Branch / commit conventions (BE)

Follow the base branch and commit format declared in `AGENTS.md`. Typical BE
conventions:

- Base branch: `development` (or `main` / `develop` per `AGENTS.md`).
- Branch: `feature/<TICKET-ID>-<kebab-slug>` (also `fix/`, `chore/`).
- Commit format (Conventional Commits + ticket ID; the exact format is
  declared in `AGENTS.md`):

  ```text
  <type>(<scope>): <imperative summary> (#<TICKET-ID>)
  ```

  where `<type>` ∈ `feat | fix | docs | style | refactor | test | chore | perf | ci | build | revert`.
- Scope typically matches the primary controller / module (e.g.
  `rif-lists`, `rosters`, `auth`, `mapper`, `handlers`).

## Manual commands (BE)

List these in the Delivery Bundle (adjust to the repo's `AGENTS.md`):

```bash
# .NET example (adjust to your stack)
dotnet restore
dotnet build ./<Solution>.sln -c Release
dotnet test  ./<Solution>.sln -s <path>/test.runsettings

# Migration (only if the ticket has schema impact AND the flow is confirmed)
dotnet ef migrations add <Name> --project <DAL> --startup-project <Web>
```

For non-.NET stacks:

```bash
# Java (Maven)
mvn -B clean verify

# Node.js server
npm ci
npm run build
npm run test

# Python
poetry install
poetry run pytest

# Go
go build ./...
go test ./...
```

## PR description additions (BE)

Add these sections after the AC coverage checklist in the base template:

```md
**Contract delta**
- `<METHOD> <path>` — policy `<name>` — 200 / 4xx …
- Request DTO: `<path>`
- Response DTO: `<path>`
- Errors follow `<global handler path>` (uniform envelope).
- OpenAPI: `<endpoint>` visible in the generated document.

**Persistence delta**
- Entities touched: `<list>`.
- Tables touched: `<list>`.
- Schema change: yes/no. If yes: migration name `<Name>` — rollback strategy.

**External integrations / events**
- <service / event / job> or `N/A`.

**Feature flag**
- Key: `<key>`, default per env.
```

## Additional rubric rows (BE)

Merge into the base rubric of `pr-handoff.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Commit format matches CI     | Conventional Commits + `#<TICKET-ID>`; CI hook (if any) would accept it.                                    |
| B2 | Contract delta included      | PR description names route, method, DTOs, policy, status codes.                                             |
| B3 | Persistence delta included   | Entities / tables / schema change stated; migration referenced if any.                                      |
| B4 | Test evidence per AC         | Every AC line lists the test(s) that assert it; integration tests cover every documented status code path.  |
| B5 | No migration was run         | The AI has not applied any migration; the exact migration command is listed for the Dev.                    |
| B6 | Pipeline gate                | PR triggers the repo's declared PR checks.                                                                  |

## Tools / MCP hooks (BE)

- Read the converged diff produced by Phase 5.
- Ticketing MCP (roadmap) — fetch/update the Work Item / Issue.
- OpenAPI MCP (roadmap) — attach a contract diff to the PR description.
