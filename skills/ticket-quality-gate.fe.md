# Skill overlay: ticket-quality-gate — FE profile

> Overlay applied on top of [`ticket-quality-gate.md`](ticket-quality-gate.md)
> when the active profile is `fe`.

## FE format check

The four immutable headings must be:

```text
## ToDo
## AC
## Links to Figma
## Notes & Obvs
```

Any renamed, reordered, or missing heading → reject.

## FE-specific repo-conventions checks

For each ticket, confirm `Notes & Obvs` (and `Links to Figma`) explicitly
name:

- Route registration with `meta.featureFlag` + `meta.requiredAccessLevels`
  (or the equivalents in the target framework).
- HTTP boundary — reads/mutations use the declared HTTP client abstraction;
  no direct `axios`/`fetch` in components/stores.
- State solution — touchpoints on the declared store solution; no ad-hoc
  singletons.
- Testing harness — tests rendered via the declared harness (`renderWithProviders`
  or equivalent).
- Design tokens used (no hard-coded colour / spacing / typography).
- Accessibility — keyboard, focus, screen-reader, contrast expectations.
- i18n keys added / reused (or `N/A`).
- Telemetry events wired (or `N/A`).

## FE landmines to sweep

- **Bypass of feature-flag or access-level gating** — reject.
- **Breaking UX change** without a versioning / migration strategy for stored
  preferences or URLs — reject / request changes.
- **New global CSS / design tokens** without a design-system slice ordered
  before this one — reject.
- **Consumed BE contract** that is not shipped and not `blocked by` a BE
  ticket — reject.
- **Component from a library version** that the repo does not use — reject.

## Additional rubric rows (FE)

Merge into the base rubric of `ticket-quality-gate.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | Figma links resolve          | Every screen link opens the intended node; node-ids present when possible.                                  |
| F2 | HTTP boundary                | Ticket names the HTTP client abstraction; no ad-hoc HTTP calls in ToDo.                                     |
| F3 | State solution               | Store touchpoints named per `AGENTS.md`.                                                                    |
| F4 | BE dependency status         | Any consumed BE contract is shipped or `blocked by` a BE ticket.                                            |
| F5 | Accessibility explicit       | Keyboard, focus, screen-reader, contrast expectations declared.                                             |
| F6 | Feature-flag / access levels | Both declared and consistent with the plan.                                                                 |

## Governance Report additions (FE)

Add these columns to the report table from the base template:

| Figma | HTTP boundary | State | BE dep | A11y |
|-------|---------------|-------|--------|------|
| ✅ | ✅ | ✅ | ✅ | ✅ |

## Tools / MCP hooks (FE)

- Read-only access to the ticket bodies.
- Diff against `templates/ticket.fe.md`.
- Figma MCP (roadmap) — validate that links resolve.
