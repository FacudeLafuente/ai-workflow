# Skill overlay: pr-handoff — FE profile

> Overlay applied on top of [`pr-handoff.md`](pr-handoff.md) when the active
> profile is `fe`.

## Branch / commit conventions (FE)

Follow the base branch and commit format declared in `AGENTS.md`. Typical FE
conventions:

- Base branch: `development` (or `main` / `develop` per `AGENTS.md`).
- Branch: `feature/<TICKET-ID>-<kebab-slug>`.
- Commit format enforced by CI (`scripts/commit-check.sh` or equivalent):

  ```text
  <type>(<scope>): <lowercase description> #<TICKET-ID>
  ```

  where `<type>` ∈ `feat | fix | docs | style | refactor | test | chore | perf | ci | build | revert`.
- Scope typically matches the module (e.g. `rif-lists`, `rosters`,
  `auth`, `design-system`).

## Manual commands (FE)

List these in the Delivery Bundle (adjust to the repo's `AGENTS.md`):

```bash
# install deps
npm install            # or: pnpm install / yarn install

# local verification before push (do not run in CI-only mode)
npm run lint:check
npm run type-check
npm run test
npm run build
```

## PR description additions (FE)

Add these sections after the AC coverage checklist in the base template:

```md
**Design & UX delta**
- Screens implemented: <name> — see Figma links from the ticket.
- Component library components used: <list>.
- Design tokens referenced: <list>.
- State variants covered: empty, loading, error, offline, soft-delete, feature-flag-off, insufficient access level.
- Accessibility: <keyboard / focus / SR / contrast summary>.

**Consumed BE contract**
- `<METHOD> <path>` — status: SHIPPED (link the BE PR/ticket) or IN-FLIGHT
  (link the BE ticket; this PR will be held until the BE change ships).

**Telemetry**
- Events emitted: <list> (or `N/A`).

**i18n**
- Keys added / reused: <list> (or `N/A`).
```

## Additional rubric rows (FE)

Merge into the base rubric of `pr-handoff.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | Commit format matches CI     | `<type>(<scope>): <lowercase>` + `#<TICKET-ID>` at the end; CI hook (if any) would accept it.               |
| F2 | Design delta included        | PR description names the Figma nodes implemented and the design-system components used.                    |
| F3 | BE contract status stated    | Consumed BE contract explicitly marked SHIPPED or IN-FLIGHT (with link).                                    |
| F4 | Test coverage per AC         | Every AC line lists the test(s) that assert it; harness is the declared one.                                |
| F5 | Bundle-size safety           | No new heavy dependency without justification; dynamic imports used where the declared strategy expects.    |

## Tools / MCP hooks (FE)

- Read the converged diff produced by Phase 5.
- Ticketing MCP (roadmap) — fetch/update the Work Item / Issue.
- Figma MCP (roadmap) — link the PR description to the design nodes.
