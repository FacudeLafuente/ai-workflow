# Skill overlay: spec-intake — FE profile

> Overlay applied on top of [`spec-intake.md`](spec-intake.md) when the active
> profile is `fe`. Adds FE-specific inputs, DoR checks, and rubric rows.

## Additional inputs (FE)

- **Figma designs** for every screen (URLs + node-ids when available).
- **Component library references** — every reused UI component (design system
  / internal library / third-party) named and linked.
- **Design System Guide** — tokens, spacing, colour, typography, iconography
  the change relies on.
- **Access levels / feature flags** — route/screen gating rules (which access
  levels can see the screen; which feature flag governs it).
- **State management touchpoints** — which stores (Pinia / Redux / NgRx /
  Zustand) the change reads or mutates; whether new store modules are needed.
- **Data layer** — which repositories / data-access hooks provide the reads
  and mutations; if new, sketch their signatures.
- **BE contract consumed** — endpoint(s) the FE calls, request/response
  shape, error codes, empty/partial-result semantics. Cite the BE ticket ID
  when this FE ticket depends on a BE change.
- **Accessibility** — keyboard navigation, screen-reader labels, focus
  management, colour contrast expectations.
- **i18n** — locales required, keys added/reused.
- **Telemetry / analytics** — event names + payload shapes to emit.
- **Performance** — bundle size / render count / LCP / route lazy-load
  expectations.

## Additional steps (FE)

- **Contract sanity-check.** For each declared screen:
  - Figma link resolves and points at the right node.
  - Every design-system component referenced exists in the current version
    of the library.
  - The BE contract consumed is either already shipped (link) or has an
    open BE ticket linked as `blocked by`.
  - Empty / loading / error / offline / soft-delete state variants are
    designed (or explicitly `N/A`).
- **Mode B validation.** Cross-check the ticket against
  `templates/ticket.fe.md` — headings must be `ToDo / AC / Links to Figma /
  Notes & Obvs`, in order.

## Additional DoR checklist rows (FE)

- [ ] Every screen has a Figma link + node-id (or `N/A — reason`).
- [ ] Every referenced design-system component exists and is version-compatible.
- [ ] Empty / loading / error / offline / soft-delete state variants
      designed or `N/A`.
- [ ] Accessibility expectations declared.
- [ ] i18n keys declared (or `N/A`).
- [ ] Telemetry events declared (or `N/A`).
- [ ] Feature flag key + required access levels declared (or `N/A`).
- [ ] Consumed BE contract declared (or `N/A` if pure FE).

## Additional rubric rows (FE)

Merge into the base rubric of `spec-intake.md`:

| # | Check                     | Pass criterion                                                                                                          |
|---|---------------------------|-------------------------------------------------------------------------------------------------------------------------|
| F1 | Design fidelity          | Every screen has a resolving Figma link + node-id; every state variant either has a design or is explicitly `N/A`.      |
| F2 | Component library sanity  | Every referenced component exists in the declared library version; no invented components.                              |
| F3 | State discipline          | The change's state read/writes are mapped to the repo's canonical store solution (per `AGENTS.md`), not ad-hoc singletons. |
| F4 | Data-layer discipline     | Reads/mutations go through the repo's HTTP client abstraction / data-access hooks — no direct HTTP-library calls.       |
| F5 | Accessibility explicit    | Keyboard, screen-reader, focus, contrast expectations are declared per screen (or `N/A — <reason>`).                    |
| F6 | Telemetry / i18n / flags  | Events, i18n keys, feature flag key, required access levels — each declared or explicitly `N/A`.                        |

## Readiness Report additions (FE)

Add these sections to the Readiness Report template from the base skill:

```md
## Design & UX

- Screens: <name> — <figma-url> (node-id)
- Components: <name> — <library ref>
- Design System: <token / pattern>
- State variants covered: empty, loading, error, offline, soft-deleted, feature-flag-off, insufficient access level.

## Data layer

- BE endpoints consumed: `<METHOD> <path>` — <request / response shape or link to BE ticket>.
- Store touchpoints: <store name(s)>.
- Data-access hooks / repositories: <name(s)>.

## Non-functional (FE)

- Feature flag: <key> — default per env.
- Required access levels: <list> or `N/A`.
- Telemetry events: <name(s) + payload>.
- Accessibility: <expectations>.
- i18n: <keys / locales>.
- Performance: <LCP / bundle-size / render-count budget>.
```

## Tools / MCP hooks (FE)

- **Figma MCP** (optional / roadmap) — validate links and node-ids
  automatically; extract component references.
- **Context7 MCP** (if declared in `AGENTS.md`) — pull framework / library
  docs when the change touches unfamiliar APIs.
