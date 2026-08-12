# Skill overlay: ticket-author — FE profile

> Overlay applied on top of [`ticket-author.md`](ticket-author.md) when the
> active profile is `fe`. Uses `templates/ticket.fe.md`.

## Heading order (immutable, FE)

```text
## ToDo
## AC
## Links to Figma
## Notes & Obvs
```

## ToDo discipline (FE)

Whenever applicable to the slice, add these atomic items to the `ToDo`:

- [ ] Route added to `<router file>` with `meta.pageTitle`,
      `meta.featureFlag`, `meta.requiredAccessLevels` (or the equivalents in
      the target framework).
- [ ] View / page component created under `<module>/views/` or `<module>/pages/`.
- [ ] Child components created under `<module>/components/`.
- [ ] Composable / hook wrapping the read/mutation created under
      `<module>/composables/` or `<module>/hooks/`.
- [ ] Repository / data-access module updated under `<module>/repositories/`
      using the declared HTTP client abstraction (never `axios`/`fetch`
      directly).
- [ ] Store slice updated under `<module>/stores/` (or `src/stores/`) — no
      module-level singletons for shared state.
- [ ] Types / models added under `<module>/types.ts` or `<module>/models/`.
- [ ] Component + store + repository tests added under co-located `__tests__/`
      folders, rendered via the declared harness.
- [ ] Design tokens used — no hard-coded colour / spacing / typography.
- [ ] Accessibility: keyboard interactions, ARIA labels, focus management,
      colour contrast verified against the tokens.
- [ ] i18n keys added / reused (or `N/A`).
- [ ] Telemetry events wired (or `N/A`).
- [ ] Build / lint / type-check / test / format command list in the PR
      description (Dev runs them).

## Links to Figma discipline

Fill the third section per this shape (always with URLs and node-ids when
available):

```md
## Links to Figma

- **Screen:** <name> — <figma-url> (node-id: <id>)
- **Screen:** <name> — <figma-url> (node-id: <id>)
- **Component (design system / library):** <name> — <lib doc or Figma URL>
- **Design System reference:** <token / pattern / spacing / colour> — <url>
- **State variants:** empty, loading, error, offline, soft-deleted — <urls or node-ids>
```

If the slice is purely FE-internal (e.g. a data-layer refactor with no visual
change), write `## Links to Figma` followed by `N/A — <reason>`. Never omit
the heading.

## Notes & Obvs discipline (FE)

Include these blocks when applicable:

- **Repo touchpoints** — one file per line (module folder, view, components,
  store, repository, router, tests).
- **BE contract consumed** — `<METHOD> <path>` with request / response shape;
  link the BE ticket if in-flight (`blocked by #<BE-ID>`).
- **Feature flag / access levels** — flag key + default per env; required
  access levels list.
- **Telemetry / analytics events** — event name + payload shape.
- **Accessibility expectations** — keyboard, screen-reader, focus,
  contrast.
- **i18n** — keys added / reused.
- **Performance** — bundle size / render-count / LCP budget.
- **Dependencies** — `blocked by #<ID>` / `blocks #<ID>`.
- **Governance / risk** — auth guard applied, PII / data exposure,
  breaking UX, backward compatibility of URLs / stored preferences.
- **Out of scope** — explicit list.
- **Manual commands** — install / dev / build / type-check / lint / test /
  format — copied from `AGENTS.md`.

## Additional rubric rows (FE)

Merge into the base rubric of `ticket-author.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | Figma links                  | Every touched screen has a resolving Figma link + node-id (or explicit `N/A — reason`).                     |
| F2 | Component library refs       | Every reused component names the library and version (or the design-system doc URL).                       |
| F3 | HTTP abstraction             | ToDo items explicitly use the declared HTTP client abstraction; no direct library calls.                    |
| F4 | State solution               | Store touchpoints named; no ad-hoc singletons.                                                              |
| F5 | Accessibility explicit       | Keyboard, screen-reader, focus, contrast expectations are declared.                                         |
| F6 | Telemetry / i18n / flags     | Events, i18n keys, feature flag, required access levels — declared or `N/A`.                                |

## Tools / MCP hooks (FE)

- Ticket template: `templates/ticket.fe.md`.
- Figma MCP (roadmap).
- Ticketing MCP (roadmap).
