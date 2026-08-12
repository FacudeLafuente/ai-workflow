# Skill overlay: repo-viability-scan — FE profile

> Overlay applied on top of [`repo-viability-scan.md`](repo-viability-scan.md)
> when the active profile is `fe`.

## Autodetection hints (FE)

The base skill autodetects `fe` when it finds `package.json` plus one of:

- `vue`, `@vue/*`, `nuxt`, `@nuxt/*`
- `react`, `next`, `remix`, `@remix-run/*`, `gatsby`
- `@angular/core`
- `svelte`, `@sveltejs/kit`
- `astro`, `solid-js`, `qwik`
- Mobile: `react-native`, `@react-native/*`, `expo`, `@ionic/*`

If both FE and BE stacks are present in the repo, the base skill halts and
asks the human to declare the profile.

## Touchpoint map to scan (FE)

Scan these paths for existing behavior and gaps. Adjust to your repo's
declared layout in `AGENTS.md`:

- **Module / feature folders** — `src/modules/*`, `src/features/*`, `apps/*`
  (module-first vs feature-first depends on `AGENTS.md`).
- **Components** — `<module>/components/`, `<module>/views/`, `<module>/pages/`.
- **Composables / hooks** — `<module>/composables/` (Vue), `<module>/hooks/` (React).
- **Stores** — `src/stores/` and `<module>/stores/` (Pinia); `src/store/`
  (Redux/Vuex); `<module>/store/` (NgRx).
- **Repositories / data-access** — `src/repositories/`, `<module>/repositories/`,
  or `<module>/data/`.
- **HTTP client abstraction** — e.g. `src/composables/api/useHttpClient.ts`,
  `src/api/http.ts` (per `AGENTS.md`).
- **Router / routes** — `src/router/`, `<module>/routes.ts`, `app/(routes)/`
  (Next.js), `app/routes/*` (Remix).
- **Auth bootstrap / guards** — `src/auth/`, route `meta` gating (feature
  flag, required access levels), main guard.
- **Error types + global error UX** — `src/models/api-errors`, `errorStore`,
  `ErrorProvider`, or the equivalents declared in `AGENTS.md`.
- **Config loader** — runtime config file(s) referenced at boot
  (`public/config.json`, `.env`, etc.).
- **Testing harness** — `src/test-utils/render.ts` (or the equivalent),
  co-located `__tests__/` folders.
- **Styling / design tokens** — CSS variables, SCSS mixins, Tailwind config,
  design-system tokens the change relies on.
- **Telemetry** — analytics wrapper, event constants file.

## Landmines (FE)

Explicitly review these repo-wide rules from `AGENTS.md` against the change:

- **HTTP boundary discipline** — all HTTP goes through the declared abstraction
  (`useHttpClient` / `apiClient` / …); direct library calls (`axios`, `fetch`)
  are forbidden in components / stores.
- **State discipline** — app-wide state lives in the declared store solution;
  module-scoped `let` / `ref` singletons are forbidden.
- **Routing discipline** — new routes go through the router's registration
  path; auth/feature-flag/access-level meta is not bypassed; `mainGuard` (or
  equivalent) is respected.
- **Import order / path aliases** — declared aliases (e.g. `@/`) are used
  instead of relative parent imports (`../../*`).
- **Paradigm** — Composition API vs Options API (Vue), Functional vs Class
  (React) — new code follows the declared paradigm.
- **Design tokens** — colours / spacing / typography come from tokens, not
  hard-coded values.
- **Accessibility** — new interactive elements have keyboard support and
  labels; contrast respects the tokens.
- **Bundle size** — new dynamic imports use the declared code-splitting
  strategy (route-level lazy load, dynamic `import()`).
- **Consumed BE contract stability** — a BE change consumed by this FE ticket
  must exist or be `blocked by` a BE ticket.
- **Breaking UX change** — visible surface changes require a design review
  and a versioning strategy for stored preferences / URLs.

## Viability Report additions (FE)

Add these rows to the Touchpoints table in the base template:

| Layer / concern | Path (examples)                                        |
|-----------------|--------------------------------------------------------|
| Module          | `src/modules/<Name>/` or `src/features/<Name>/`        |
| View / page     | `<module>/views/*`, `<module>/pages/*`                  |
| Component       | `<module>/components/*`                                 |
| Composable / hook | `<module>/composables/*`, `<module>/hooks/*`          |
| Store           | `<module>/stores/*` or `src/stores/*`                   |
| Repository / data | `<module>/repositories/*` or `src/repositories/*`     |
| Router          | `<module>/routes.ts`, `src/router/index.ts`             |
| Auth / guards   | `src/auth/*`, `mainGuard`, route `meta`                 |
| Errors          | `src/models/api-errors`, error store / provider         |
| Testing harness | `src/test-utils/render.ts`                              |
| Design tokens   | `src/styles/tokens.*`, `tailwind.config.*`              |

And add a **BE dependency** section:

```md
## BE dependency

- `<METHOD> <path>` — status: SHIPPED / IN-FLIGHT / MISSING.
  - If IN-FLIGHT or MISSING: link the BE ticket and mark this slice as
    `blocked by #<BE-TICKET-ID>`.
```

## Additional rubric rows (FE)

Merge into the base rubric of `repo-viability-scan.md`:

| # | Check                    | Pass criterion                                                                                              |
|---|--------------------------|-------------------------------------------------------------------------------------------------------------|
| F1 | HTTP boundary            | Every HTTP call in the plan uses the declared abstraction; no direct library calls.                         |
| F2 | State discipline         | Every read/write on app-wide state uses the declared store solution.                                        |
| F3 | Route registration       | New routes wired through the router with the declared `meta` (feature flag, access levels).                 |
| F4 | Design fidelity mapping  | Every screen in the plan maps to a Figma node and a resolvable design-system component set.                 |
| F5 | BE dependency explicit   | Every consumed BE contract is either shipped or explicitly `blocked by` a BE ticket.                        |
| F6 | Accessibility touchpoints | New interactive components have declared keyboard, focus, contrast, and screen-reader expectations.        |

## Tools / MCP hooks (FE)

- Repository search / read.
- Language server (Vue-tsc, TypeScript, ESLint) for symbol references when
  available.
- **Figma MCP** (optional / roadmap).
- **Context7 MCP** for framework / library docs.
