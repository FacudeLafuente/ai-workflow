# Skill overlay: implementation-loop — FE profile

> Overlay applied on top of [`implementation-loop.md`](implementation-loop.md)
> when the active profile is `fe`.

## Implementation order (FE)

Implement the slice top-down through these layers (skip a layer if the
slice does not touch it):

1. **Types / models** — declare or update domain types in `<module>/types.ts`
   or `<module>/models/`. Types are the contract other layers depend on.
2. **Repository / data-access hook** — under `<module>/repositories/` or
   `<module>/data/`. Always call the declared HTTP client abstraction (never
   `axios` / `fetch` directly).
3. **Store slice** — under `<module>/stores/` (Pinia / Redux / NgRx /
   Zustand). No ad-hoc singletons for shared state.
4. **Composables / hooks** — under `<module>/composables/` (Vue) or
   `<module>/hooks/` (React). Encapsulate reads/mutations + local UI logic.
5. **Child components** — under `<module>/components/`. Small, dumb, typed
   props/emits.
6. **View / page component** — under `<module>/views/` or `<module>/pages/`.
   Composes children + wires the composable/hook. Handles states: empty,
   loading, error, offline, soft-delete, feature-flag-off, insufficient
   access level.
7. **Route registration** — in `<module>/routes.ts` (or `app/(routes)/…` for
   Next.js, `app/routes/*` for Remix). Add `meta.pageTitle`,
   `meta.featureFlag`, `meta.requiredAccessLevels`.
8. **Tests** — co-located in `__tests__/`. Render components via the declared
   harness. Cover every AC line and every non-`N/A` state from the Phase 0
   matrix.
9. **Design tokens** — replace any placeholder styling with declared tokens;
   verify contrast against the tokens.

## Discipline (FE)

- **HTTP boundary** — every HTTP call goes through the declared abstraction;
  the abstraction handles auth token injection, retry, and error typing.
- **State discipline** — reads/writes on app-wide state go through the
  declared store solution.
- **Errors** — throw / propagate typed errors (`BadRequest`, `Forbidden`,
  `NotFound`, `Unauthorized`, `NetworkError`, `Timeout`, `Unknown` — per
  `AGENTS.md`). The default UX is the global error handler; opt out per
  request only when the caller handles the error inline.
- **Auth guard** — new routes carry the declared `meta` gating; don't bypass
  the main guard by removing meta keys.
- **Accessibility** — keyboard navigation, focus management, ARIA labels,
  colour contrast — verified against design tokens.
- **Bundle discipline** — route components loaded via dynamic `import()` per
  the declared code-splitting strategy.
- **No commands** — do not run `npm run dev`, `npm run build`, `npm run test`,
  `npm run lint*`, `git *`. List them in the delivery summary.

## Additional rubric rows (FE)

Merge into the base rubric of `implementation-loop.md`:

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | HTTP boundary                | No direct `axios`/`fetch` calls added; all HTTP through the declared abstraction.                           |
| F2 | State discipline             | New shared state lives in the declared store solution; no module-level mutable singletons.                  |
| F3 | Auth guard                   | New routes carry `meta.featureFlag` + `meta.requiredAccessLevels` (or equivalents); guard not bypassed.     |
| F4 | Typed errors                 | Errors are typed; global error UX honored unless the ticket explicitly opts a request out.                  |
| F5 | Design tokens                | Colour / spacing / typography come from tokens; no hard-coded values added.                                 |
| F6 | Accessibility                | Keyboard, focus, ARIA labels, contrast covered per the ticket's expectations.                               |
| F7 | Test harness                 | Component tests rendered via the declared harness; no bespoke provider stubs that diverge from production.  |

## Implementation Handoff Report additions (FE)

Add a **Design & UX delta** block:

```md
## Design & UX delta

- Screens implemented: <name> — see Figma link in the ticket.
- Component library components used: <list>.
- Design tokens referenced: <list>.
- State variants implemented: empty, loading, error, offline, soft-delete, feature-flag-off, insufficient access level.
- Accessibility verified: <keyboard / focus / SR / contrast>.
```

## Tools / MCP hooks (FE)

- Repo read/write (edit files under the module folder + `src/router/`,
  `src/stores/`, `src/repositories/`, tests).
- Subagent for the reviewer step.
- Figma MCP (roadmap) — cross-check implementation against design nodes.
- **No** command execution (`npm`, `pnpm`, `yarn`, `git`).
