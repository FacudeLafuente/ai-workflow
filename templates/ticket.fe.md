<!--
=============================================================================
 IMMUTABLE TICKET FORMAT — ASDD (Agentic Spec-Driven Development) — FE profile
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
 - Format-exact:   do not rename `ToDo`, `AC`, `Links to Figma`, `Notes & Obvs`.
 - Self-contained: an implementer must be able to build the slice from this
                   ticket alone (no hidden context in chat/thread history).
 - Repo conventions: honor the rules declared in the consumer repo's AGENTS.md
                   (module layout, state management, HTTP client abstraction,
                   auth guards, styling / design tokens, testing harness).

 The four headings for the FE profile are:
   ## ToDo
   ## AC
   ## Links to Figma
   ## Notes & Obvs
=============================================================================
-->

# <TICKET-ID> — <Short imperative title>

## ToDo

<!--
Atomic, verifiable checklist items. Each item is something a reviewer can tick
off from the diff (a file created, a component added, a route wired, a store
action added, a Vue Query hook created, a test class added). No verbs like
"investigate" or "consider" — those belong in Phase 1, not in a ticket.
-->

- [ ] <atomic action 1>
- [ ] <atomic action 2>
- [ ] Route added / gated by `meta.featureFlag` and `meta.requiredAccessLevels`
      (or the equivalent in the target framework)
- [ ] Data layer wired via the repo's HTTP client abstraction (never call the
      raw HTTP library directly)
- [ ] State stored in the canonical store solution (Pinia / Redux / Zustand /
      NgRx — as declared in AGENTS.md)
- [ ] Errors typed and routed through the global error UX (or opted out
      explicitly per the AGENTS.md convention)
- [ ] Unit tests added under the correct `__tests__/` folder; rendered via the
      repo's test harness (`renderWithProviders` / equivalent)
- [ ] Build / lint / test / format command list included in the PR description
      (Dev runs them)

## AC

<!--
Given / When / Then. Each line traces to a Business/Product criterion.
Always include an explicit "Edge cases / states covered" line so the loop
verifier can enforce full coverage.
-->

- **Given** <precondition> **When** <action> **Then** <observable outcome>.
- **Given** <precondition> **When** <action> **Then** <observable outcome>.
- **Edge cases / states covered:** empty, loading, error (network / server /
  validation), unauthorized (401), forbidden (403), not-found (404), offline,
  soft-deleted source data, feature-flag off, insufficient access level, race
  between two user actions. Mark any that do **not** apply with
  `N/A — <reason>`; never silently omit.

## Links to Figma

<!--
Every screen the ticket touches must be linked. Include node-ids when
available; screenshots as backup are fine but do not replace the link.
When the ticket is a BE-only or infra-only slice, write `N/A` explicitly
and justify in `Notes & Obvs`.
-->

- **Screen:** <name> — <figma-url> (node-id: <id>)
- **Screen:** <name> — <figma-url> (node-id: <id>)
- **Component (design system / component library):** <name> — <figma-url or lib doc url>
- **Design System reference:** <token / pattern / spacing / colour / typography> — <url>
- **Empty / loading / error states referenced:** <urls / node-ids>

## Notes & Obvs

<!--
Everything a reviewer needs that is not part of the design: touchpoints,
dependencies, BE contract deltas (for FE tickets that consume a BE change),
governance/risk, out-of-scope, manual commands.
-->

- **Repo touchpoints:**
  - `<path to module folder or feature folder>`
  - `<path to view / component>`
  - `<path to store>`
  - `<path to repository / data-access hook>`
  - `<path to route definition>`
  - `<path to test file>`
- **BE contract consumed:** `<METHOD> <path>` — request / response shape (or
  link to the BE ticket in `Contracts & Specs`). Write `N/A` if pure FE.
- **Feature flag / access level:**
  - Flag key: `<key>` (default per environment).
  - Required access levels: `<list>` (or `N/A`).
- **Telemetry / analytics events:** `<event name>` with payload `<shape>` (or
  `N/A`).
- **Accessibility:** keyboard navigation, screen-reader labels, focus
  management, colour contrast — describe expectations or `N/A`.
- **i18n:** keys added / reused (or `N/A`).
- **Performance:** budget (LCP / bundle-size / render-count), memoization
  strategy — or `N/A`.
- **Dependencies:** `blocked by #<ID>` / `blocks #<ID>` (or `none`).
- **Governance / risk:**
  - Auth guard: <route guard applied and not relaxed>.
  - PII / data exposure: <describe> or `N/A`.
  - Breaking UX change: <describe> or `N/A`.
  - Backward compatibility of stored preferences / URLs: <describe> or `N/A`.
- **Out of scope:** <list explicitly>.
- **Manual commands the Dev will run (AI never runs these):**
  - `<install command>` (e.g. `npm install`, `pnpm install`, `yarn install`)
  - `<lint command>` (e.g. `npm run lint:check`)
  - `<type-check command>` (e.g. `npm run type-check`)
  - `<test command>` (e.g. `npm run test`)
  - `<build command>` (e.g. `npm run build`)
