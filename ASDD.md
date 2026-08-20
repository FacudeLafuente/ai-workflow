# ASDD — Agentic Spec-Driven Development

> **Spec → Scan → Plan → Ticket → Build → PR**
>
> Canonical, platform-neutral and framework-neutral workflow for AI-assisted
> work in any repository. ASDD owns the **workflow** (how work moves from a
> business idea to a merged PR); it delegates **code conventions** to the
> consumer repo's `AGENTS.md`.

---

## Table of contents

1. [Principles](#principles)
2. [Profiles](#profiles)
3. [Flow overview](#flow-overview)
4. [Per-phase definitions](#per-phase-definitions)
5. [The self-verification loop](#the-self-verification-loop)
6. [Skills catalog](#skills-catalog)
7. [Ticket format](#ticket-format)
8. [How to wire the skills per platform](#how-to-wire-the-skills-per-platform)
9. [File structure](#file-structure)
10. [Optional architectural patterns](#optional-architectural-patterns)
11. [Roadmap](#roadmap)
12. [Glossary](#glossary)

---

## Principles

1. **Spec first, always.** No Acceptance Criteria + use cases + contracts ⇒
   the flow does not start. The spec is the contract.
2. **Humans own the gates.** The AI proposes and builds; the Developer approves
   at the critical points (tickets, review/testing, PR).
3. **Loop until convergence.** No artifact is delivered on the first try; the
   AI self-reviews until it can no longer be improved.
4. **Root cause, not shortcuts.** Silencing warnings/tests, narrowing scope to
   dodge a check, or hiding debt is forbidden. Anything unresolved is escalated
   explicitly.
5. **Granular and parallelizable.** Each ticket = one shippable slice → one
   focused PR.
6. **Immutable ticket format.** The four headings never change; only the name
   of heading #3 depends on the active profile (see [Ticket format](#ticket-format)).
7. **Two entry points.** Start from **raw business context** or from an
   **already-groomed ticket**; the scope changes, not the methodology.
8. **Console commands: humans only.** The AI **never** runs build / test /
   migrations / `git push` or similar. It prepares and lists the exact commands
   in the final summary; the Developer runs them (security, permissions,
   credentials).
9. **Single Source of Truth (SSOT).** Every rule, template, contract, and
   artifact has exactly one canonical location. Skills base + overlay,
   `AGENTS.md`, ticket templates, the loop prompt, and the per-phase
   Readiness/Viability/Plan/Ticket reports — none of them duplicates content
   that already lives elsewhere. If two files could carry the same rule, the
   correct one carries it and the other **links**. Duplication is a defect
   the self-verification loop rejects (see the SSOT rubric row in
   [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md)).

---

## Profiles

ASDD ships with two profiles that share the entire flow and diverge only in
the domain-specific rubrics and templates. A third value, `mixed`, is not a
profile of its own — it declares that the repo hosts both FE and BE code
and defers the profile choice to each ticket.

| Profile | Typical stacks | Heading #3 of the ticket | Conventions source |
|---|---|---|---|
| `fe` — Frontend | Vue / React / Angular / Svelte SPAs; mobile | `Links to Figma` | `AGENTS.md` at repo root |
| `be` — Backend  | .NET, JVM, Node, Python, Go APIs and services | `Contracts & Specs` | `AGENTS.md` at repo root |
| `mixed` — FE + BE in the same repo | Monorepo (`apps/web` + `apps/api`), or monolith with FE + BE intermingled (ASP.NET MVC + Razor, Next.js app + API routes, Django + templates, Rails, Laravel) | resolved per ticket (see below) | `AGENTS.md` at repo root, plus optional per-package `AGENTS.md` (closest-wins) |

### Profile selection (in order of precedence)

1. **Frontmatter in `AGENTS.md`:**

   ```md
   ---
   asdd_profile: fe   # or "be", or "mixed"
   ---
   ```

2. **`asdd/config.yml`** (for monorepos with per-package overrides —
   see `config.example.yml`).

3. **Autodetection** by `repo-viability-scan`:
   - `package.json` + one of `vue|react|angular|svelte|@vue/*|next|nuxt|remix|@angular/*` → `fe`.
   - `.csproj`, `.sln`, `pom.xml`, `build.gradle`, `go.mod`, `Cargo.toml`,
     `pyproject.toml`, `Gemfile`, `composer.json` → `be`.
   - Both FE and BE indicators detected in the same repo →
     `mixed` if the user opts in, otherwise the skill halts and asks the
     human to pick a profile explicitly.

### Effective profile for a `mixed` repo

When `asdd_profile: mixed` is declared, ASDD does **not** load a
combined ruleset. The effective profile is resolved **per ticket** in
this order of precedence:

1. **Per-package `AGENTS.md`** — if the ticket's touchpoints all live
   under one folder that carries its own `AGENTS.md` with
   `asdd_profile: fe` or `asdd_profile: be`, that profile wins
   (closest-wins rule).
2. **`asdd/config.yml` → `packages:`** — same idea, without a physical
   per-package `AGENTS.md`.
3. **Ticket frontmatter** — the ticket declares its own profile:

   ```md
   ---
   profile: fe   # or "be"
   ---

   # <TICKET-ID> — <Short imperative title>
   ...
   ```

If none of the three resolve to a single profile, the current skill
**halts** and asks the human to pick `fe` or `be` before continuing. No
skill runs against a `mixed` repo without a resolved effective profile.

The active profile determines:

- Which **overlay** file each skill reads (`<skill>.<profile>.md`).
- Which **ticket template** the ticket-author uses (`templates/ticket.<profile>.md`).
- Which **autodetected conventions** are cross-checked against `AGENTS.md`.

---

## Flow overview

```mermaid
flowchart TD
    A[Entry A: Raw business context] --> P0
    B[Entry B: Groomed Work Item] --> P0

    P0{{Phase 0 — Intake Gate\nDefinition of Ready}}
    P0 -- BLOCKED --> HUMAN0[Escalate: ask Business/Product]
    P0 -- READY --> P1

    P1{{Phase 1 — Discovery & Viability}}
    P1 -- RED --> HUMAN1[Escalate: decision required]
    P1 -- AMBER --> P2
    P1 -- GREEN --> P2

    P2[Phase 2 — Planning: Action Plan\nvertical slices + dep graph] --> P3
    P3[Phase 3 — Ticketing\nfixed 4-heading format] --> P4

    P4{{Phase 4 — Human Ticket Review\n(Governance Gate)}}
    P4 -- Changes requested --> P3
    P4 -- Approved --> P5

    P5[Phase 5 — Implementation per ticket\nself-verification loop]
    P5 --> P6

    P6{{Phase 6 — Manual Dev Review & Testing}}
    P6 -- Changes requested --> P5
    P6 -- Approved --> P7

    P7[Phase 7 — Delivery\nbranch → commit → push → PR\n(Dev runs git; AI proposes)]
    P7 --> PRR[[PR Review & Merge]]
```

| # | Phase                                | Owner              | Gate                    | Skill (base)                                                          |
|---|--------------------------------------|--------------------|-------------------------|-----------------------------------------------------------------------|
| 0 | Intake Gate (Definition of Ready)    | AI + Business/Prod | DoR (`READY`/`BLOCKED`) | [spec-intake](skills/spec-intake.md)                                  |
| 1 | Discovery & Viability                | AI                 | `GREEN`/`AMBER`/`RED`   | [repo-viability-scan](skills/repo-viability-scan.md)                  |
| 2 | Planning (Action Plan)               | AI                 | Complete plan           | [action-plan-builder](skills/action-plan-builder.md)                  |
| 3 | Ticketing                            | AI                 | Fixed 4-heading format  | [ticket-author](skills/ticket-author.md)                              |
| 4 | Human Ticket Review & Governance     | **Human**          | Tickets approved        | [ticket-quality-gate](skills/ticket-quality-gate.md)                  |
| 5 | Implementation per ticket            | AI                 | Loop convergence        | [implementation-loop](skills/implementation-loop.md)                  |
| 6 | Manual Dev Review & Testing          | **Human**          | Dev approval            | —                                                                     |
| 7 | Delivery (branch → commit → push → PR) | Dev + AI          | PR open & linked        | [pr-handoff](skills/pr-handoff.md)                                    |

Every skill file lives at `skills/<name>.md` (neutral base) and is augmented at
runtime by `skills/<name>.<profile>.md` (profile overlay). See
[Skills catalog](#skills-catalog) for the overlay index.

---

## Per-phase definitions

### Phase 0 — Intake Gate (Definition of Ready)

**Owner:** AI + Business/Product. **Skill:** `spec-intake` + profile overlay.

Guarantees all inputs exist before any work.

- **Mode A — Raw business context:**
  - Acceptance Criteria from Business/Product.
  - Use cases / states / needs.
  - **Contract expectations** — profile-specific (BE: HTTP routes + DTOs +
    status codes + error envelope mapping; FE: screens + Figma nodes +
    component library references + design system tokens).
  - **Data/schema touched** — entities, tables, models, stores.
  - **Integrations** — external services, events/messages, background jobs.
  - **Non-functional** — auth policy, performance, idempotency, pagination,
    concurrency, PII / exposure rules, feature flag.
- **Mode B — Already-groomed ticket:** the ticket has the fixed 4 sections and
  is the spec; scope is fixed; Phase 3 is validated (not re-authored).

**Outputs:** *Readiness Report* (`READY` / `BLOCKED`), ACs normalized into
`Given/When/Then`, and a states/edge-cases matrix (empty, not-found,
unauthorized/forbidden, validation error, conflict, rate-limit, partial failure,
idempotent retry, concurrent modification — plus profile-specific states like
loading, offline, soft-delete).

**Gate (DoR):** any missing input ⇒ `BLOCKED`. Nothing is assumed or invented.

### Phase 1 — Discovery & Viability

**Owner:** AI. **Skill:** `repo-viability-scan` + profile overlay.

Verify the real, current state of the repo and feasibility.

- **Detect the profile** if not declared (see [Profiles](#profiles)).
- **Locate touchpoints** — profile-specific paths (controllers/services/repos
  for BE; components/stores/composables/repositories for FE).
- Compare desired vs current behavior; list gaps.
- **Flag landmines and constraints** from `AGENTS.md` (layering, auth, error
  envelope, state management, styling conventions, etc.).
- **Rating:** `GREEN` (no unknowns), `AMBER` (open questions but not blocking),
  `RED` (blocker — schema change without migrations flow, missing auth policy,
  breaking API/UI change, missing critical design). `RED` escalates to the
  human with the exact decision required.

### Phase 2 — Planning (Action Plan)

**Owner:** AI. **Skill:** `action-plan-builder` + profile overlay.

Decompose into **vertical slices** (each independently shippable). Per slice:

- Objective, AC subset, touchpoints, states/edges covered, risks, dependencies
  (`blocks` / `blocked-by`).
- Build an **acyclic dependency graph**; mark parallelizable slices; assign
  orchestration (agent / subagent / skill / MCP + where a loop is required).
- **Right-size each slice to a small, focused PR** (split if too large).
- 100% AC coverage; no orphan or duplicate AC across slices.

### Phase 3 — Ticketing

**Owner:** AI. **Skill:** `ticket-author` + profile overlay.

Turn each slice into a Work Item using the fixed format from
`templates/ticket.<profile>.md`. Empty section ⇒ `N/A`. Encode dependencies
in *Notes & Obvs*. Mode B only validates/links the existing ticket (splitting
into PR-sized tickets only if the plan requires).

### Phase 4 — Human Ticket Review & Governance

**Owner:** Human, AI-assisted. **Skill:** `ticket-quality-gate` + profile overlay.

Checklist per ticket:

- Granular (one slice), PR-sized, parallelizable (explicit deps), governable
  (single stage), format-exact, traceable (every AC → a Business/Product
  criterion), self-contained.
- Approved → development starts; changes → back to Phase 3.

### Phase 5 — Implementation per ticket

**Owner:** AI. **Skill:** `implementation-loop` + profile overlay.

Implement one slice with the self-verification loop until convergence.

- Respect the layering and conventions declared in `AGENTS.md` (per profile).
- Cover all states/edges from the Phase 0 matrix.
- **Never run commands.** Prepare the exact build/test/migration commands and
  list them for the Developer.
- Deliver: code diff + iteration log + (if any) *Unresolved & Escalations*.
- Tickets without dependencies may run in parallel.

### Phase 6 — Manual Dev Review & Testing

**Owner:** Human.

The Developer reviews and runs the tests locally (using the exact commands the
AI listed); approves or returns to Phase 5.

### Phase 7 — Delivery (PR)

**Owner:** Dev + AI. **Skill:** `pr-handoff` + profile overlay.

- Branch: `feature/<TICKET-ID>-<slug>` (or `fix/`, `chore/`), branched off the
  base branch declared in `AGENTS.md` (typical: `development`, `main`, `develop`).
- Conventional commits referencing the ticket ID (e.g.
  `feat(<scope>): <imperative summary> (#<TICKET-ID>)`).
- PR targets the base branch, linked to the Work Item / Issue, with an AC
  coverage checklist.
- Focused diff (one ticket per PR).
- **No `--force` on shared branches, no `--no-verify`.**
- The **Developer** runs git commands. The AI only lists the exact suggested
  commands and the PR title/description.

---

## The self-verification loop

Every AI artifact follows the engine defined in
[`prompts/loop-self-verify.md`](prompts/loop-self-verify.md):

**Draft → Review (fresh subagent, evaluates ONLY against rubric + AC) → Fix
root cause → Repeat.**

- Stop on **Convergence** — all rubric checks pass and a full pass yields zero
  new improvements.
- Stop on **Budget** — `MAX_ITERATIONS` (default 5) → escalate with an
  *Unresolved & Escalations* section; **do not** hand off as complete.

**Anti-patterns (rejected):**

- Infinite loops with no convergence signal.
- "Green by deletion" — passing checks by removing the failing scenario.
- Hidden debt reported as complete.
- Reviewer bias — the reviewer must be a fresh subagent, not the author
  re-reading its own draft.
- Command execution by the AI (build/test/`git push`/migrations).

Each phase overrides the default rubric with phase-specific criteria (see the
skill file). The profile overlay adds/refines rubric rows.

---

## Skills catalog

| Skill                                                            | Phase | Purpose                                                              | Overlays                                                                                                                                   |
|------------------------------------------------------------------|-------|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| [spec-intake](skills/spec-intake.md)                             | 0     | Normalize inputs into a Readiness Report; enforce DoR                | [fe](skills/spec-intake.fe.md) / [be](skills/spec-intake.be.md)                                                                             |
| [repo-viability-scan](skills/repo-viability-scan.md)             | 1     | Map touchpoints; rate GREEN/AMBER/RED; detect profile if undeclared   | [fe](skills/repo-viability-scan.fe.md) / [be](skills/repo-viability-scan.be.md)                                                             |
| [action-plan-builder](skills/action-plan-builder.md)             | 2     | Slice into PR-sized vertical slices with a dependency graph          | [fe](skills/action-plan-builder.fe.md) / [be](skills/action-plan-builder.be.md)                                                             |
| [ticket-author](skills/ticket-author.md)                         | 3     | Emit Work Items in the fixed 4-heading format for the active profile | [fe](skills/ticket-author.fe.md) / [be](skills/ticket-author.be.md)                                                                         |
| [ticket-quality-gate](skills/ticket-quality-gate.md)             | 4     | Assist the human governance gate over the tickets                    | [fe](skills/ticket-quality-gate.fe.md) / [be](skills/ticket-quality-gate.be.md)                                                             |
| [implementation-loop](skills/implementation-loop.md)             | 5     | Build the slice; drive the self-verification loop                    | [fe](skills/implementation-loop.fe.md) / [be](skills/implementation-loop.be.md)                                                             |
| [pr-handoff](skills/pr-handoff.md)                               | 7     | Propose the exact git/PR commands and PR title/description           | [fe](skills/pr-handoff.fe.md) / [be](skills/pr-handoff.be.md)                                                                               |

Reusable prompt: [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md).
Ticket templates: [`templates/ticket.fe.md`](templates/ticket.fe.md),
[`templates/ticket.be.md`](templates/ticket.be.md).

**How overlays are applied.** At the start of each phase, the agent loads:

1. The base skill (`skills/<name>.md`) — profile-agnostic Purpose, Steps,
   default Loop, DoD.
2. The profile overlay (`skills/<name>.<profile>.md`) — profile-specific
   Inputs, Steps additions, Rubric rows, Templates, Tools/MCP hooks.

The overlay is **additive**: it adds new rubric rows and refines steps; it
never removes principles. The base file defines the invariants; the overlay
defines the profile's discipline.

---

## Ticket format

The four headings are **immutable** and must appear in this exact order.
Empty section ⇒ `N/A`. Only heading #3 changes name per profile:

| # | Heading (FE profile) | Heading (BE profile) |
|---|----------------------|----------------------|
| 1 | `ToDo`               | `ToDo`               |
| 2 | `AC`                 | `AC`                 |
| 3 | `Links to Figma`     | `Contracts & Specs`  |
| 4 | `Notes & Obvs`       | `Notes & Obvs`       |

- FE ticket template: [`templates/ticket.fe.md`](templates/ticket.fe.md).
- BE ticket template: [`templates/ticket.be.md`](templates/ticket.be.md).

**Do not** invent a fifth heading. Anything that does not fit the four goes
under `Notes & Obvs`. This is enforced by `ticket-quality-gate`.

---

## How to wire the skills per platform

Skill files under `skills/` are **platform-neutral Markdown**. Register them
per tool. **The source of truth for each skill is the file under
`asdd/ai-workflow/skills/`**. Wrappers only point at it. Do not duplicate content.

Ready-to-copy shim templates live in [`templates/`](templates/):

- **GitHub Copilot.** Copy
  [`templates/copilot-instructions.md.example`](templates/copilot-instructions.md.example)
  to `.github/copilot-instructions.md`. Inline reference:

  ```md
  This repo uses ASDD from `asdd/ai-workflow/`. Start by reading `asdd/ai-workflow/ASDD.md`.
  For each phase load `asdd/ai-workflow/skills/<name>.md` plus the overlay for the
  active profile: `asdd/ai-workflow/skills/<name>.<profile>.md` where `<profile>`
  is read from `AGENTS.md` frontmatter (`asdd_profile:`) or from
  `asdd/config.yml`.
  ```

- **Claude Code.** Two pieces:

  1. Copy [`templates/CLAUDE.md.example`](templates/CLAUDE.md.example) to
     `CLAUDE.md` at the repo root — this is a shim so Claude Code loads
     `AGENTS.md`.
  2. For each skill, create a thin wrapper at
     `.agents/skills/<name>/SKILL.md`:

     ```md
     ---
     name: <name>
     description: <one-line purpose>
     ---
     See `asdd/ai-workflow/skills/<name>.md` and `asdd/ai-workflow/skills/<name>.<profile>.md`
     where `<profile>` is declared in `AGENTS.md` frontmatter or
     `asdd/config.yml`.
     ```

- **Codex / Cursor.** Copy
  [`templates/cursor-rules.md.example`](templates/cursor-rules.md.example)
  to `.cursor/rules/asdd.md` (or the equivalent rules file for your
  Cursor version).

---

## File structure

```text
asdd/                                   ← manual buffer, ignored by your project's git
├── config.yml                          ← optional; consumer-owned overrides
└── ai-workflow/                        ← clone of this repo (its own git)
    ├── README.md
    ├── ASDD.md                            ← this file (workflow spec)
    ├── AGENTS.md                          ← canonical AGENTS.md spec (shared base)
    ├── CLAUDE.md                          ← shim: read AGENTS.md first
    ├── config.example.yml                 ← template for asdd/config.yml
    ├── templates/
    │   ├── AGENTS.md.fe.example           ← Frontend AGENTS.md template
    │   ├── AGENTS.md.be.example           ← Backend AGENTS.md template
    │   ├── CLAUDE.md.example              ← Claude Code root shim
    │   ├── copilot-instructions.md.example ← GitHub Copilot shim (→ .github/)
    │   ├── cursor-rules.md.example        ← Cursor rules shim (→ .cursor/rules/)
    │   ├── ticket.fe.md                   ← Frontend ticket
    │   └── ticket.be.md                   ← Backend ticket
    ├── prompts/
    │   └── loop-self-verify.md
    └── skills/
        ├── spec-intake.md                 ← Phase 0 base + .fe / .be overlays
        ├── repo-viability-scan.md         ← Phase 1 base + .fe / .be overlays
        ├── action-plan-builder.md         ← Phase 2 base + .fe / .be overlays
        ├── ticket-author.md               ← Phase 3 base + .fe / .be overlays
        ├── ticket-quality-gate.md         ← Phase 4 base + .fe / .be overlays
        ├── implementation-loop.md         ← Phase 5 base + .fe / .be overlays
        └── pr-handoff.md                  ← Phase 7 base + .fe / .be overlays
```

---

## Optional architectural patterns

ASDD owns the **workflow** — it is deliberately silent about the code's
**architecture** so it stays framework-agnostic. Consumers may adopt any
compatible architectural pattern in their repo; the pattern lives in
`AGENTS.md` (per profile), not in the ASDD skills.

This section catalogs patterns known to compose well with ASDD. They are
**optional and non-invasive** — nothing in ASDD requires them.

### FSD — Feature-Sliced Design (Frontend)

**What it is.** A community-driven architectural methodology for organizing
frontend codebases into **layers → slices → segments** so features stay
isolated, dependencies flow one direction, and business meaning is visible
in the folder tree. Popular with Vue / React / Angular / Svelte SPAs.

**Layers** (top-down; a layer may only import from layers below it):

1. `app` — application bootstrap (providers, router root, global styles).
2. `processes` — cross-page flows (auth flow, checkout flow). *Optional.*
3. `pages` — routed screens; each page composes widgets + features.
4. `widgets` — self-contained composite blocks used across pages
   (headers, sidebars, complex forms).
5. `features` — units of user-facing functionality with side effects (login
   form, add-to-cart button).
6. `entities` — business entities and their read/write UI + logic (User,
   Product, RIF-List).
7. `shared` — reusable, business-agnostic primitives (UI kit, lib
   utilities, API client).

**Slices** — inside each layer, a folder per domain / feature name (e.g.
`entities/user/`, `features/login/`). Slices in the same layer must not
import from each other; if they need to share, it moves to a lower layer
(usually `shared` or `entities`).

**Segments** — inside each slice, a fixed set of technical roles:

- `ui/` — components and views.
- `api/` — repositories, network calls (via the declared HTTP boundary).
- `model/` — state (store slice), types, business logic.
- `lib/` — helpers scoped to the slice.
- `config/` — constants, feature flags.

**How it composes with ASDD.**

- **Workflow is unchanged.** All 8 ASDD phases run identically. The
  ticket format, the gates, and the self-verification loop do not change.
- **Vertical slices in Phase 2 map cleanly to FSD slices.** An
  `action-plan-builder` slice ("add a `<feature>` for `<entity>`") becomes
  a diff inside one or two FSD slices, with predictable segments touched
  (`ui`, `model`, `api`).
- **`repo-viability-scan.fe` touchpoints re-map** to FSD paths
  (`src/app/`, `src/pages/`, `src/widgets/`, `src/features/`,
  `src/entities/`, `src/shared/`) instead of `src/modules/`.
- **`AGENTS.md.fe` conventions extend** with two rules that FSD enforces
  and ASDD respects:
  - Import direction: a layer may only import from layers below it. New
    `ToDo` items list which layers each file lives in.
  - Slice isolation: two slices in the same layer cannot import each other.

**When to adopt FSD.**

- Green-field frontend, or a scoped refactor of an existing FE with clear
  domain boundaries.
- Teams that want a shared vocabulary of *layer* / *slice* / *segment*
  across projects.

**When NOT to adopt FSD.**

- The repo has a working, well-understood module-first or feature-first
  layout — a wholesale move is a big refactor with real cost.
- Very small apps where 3–4 layers add more overhead than they save.

**How to activate.**

- In the consumer's `AGENTS.md.fe`, replace or extend the
  `## Project Layout — Non-Obvious Bits` section with the FSD layer table,
  the slice-isolation rule, and the import-direction rule. **Do not**
  duplicate the FSD spec here; link to the official
  [`feature-sliced.design`](https://feature-sliced.design) reference.
- Update `Code Conventions` with the "import direction" rule and any
  ESLint plugin used to enforce it
  (e.g. `@feature-sliced/eslint-config`).
- Update the `## Project Layout` in every FE ticket's `Notes & Obvs`
  touchpoints to use FSD paths.

**Reference:** <https://feature-sliced.design>.

### Other patterns (placeholders — add as they get adopted)

- **Clean Architecture / Hexagonal (BE).** Compatible; the layering
  section of `AGENTS.md.be` already covers Controller → Service →
  Repository, which maps to the Application / Domain / Infrastructure
  cut. Document adoption details here when a repo standardizes on it.
- **DDD Tactical (BE).** Compatible; goes under `Domain Core` and
  `Code Conventions → Layering` in the consumer's `AGENTS.md`.
- **Atomic Design (FE).** Compatible with or independent of FSD; goes
  under `Project Layout` in the consumer's `AGENTS.md.fe`.

---

## Roadmap

- **Ticketing MCP (per profile).** Auto-create Work Items / GitHub Issues /
  Jira tickets directly from `ticket-author`.
- **Design MCP (FE profile).** Validate Figma links and component-library
  references automatically in `spec-intake.fe`.
- **OpenAPI MCP (BE profile).** Diff proposed contracts against the live
  Swagger/OpenAPI document in `spec-intake.be`.
- **CLI.** `npx asdd init` to scaffold `AGENTS.md` + platform wrappers.
- **Autodetection tuning.** Extend `repo-viability-scan` heuristics for mobile
  (React Native, Flutter), full-stack frameworks (Next.js, Remix, Nuxt), and
  edge/serverless.
- **Additional profiles.** `data` (dbt / airflow / notebooks), `infra`
  (terraform / helm / IaC) if the community asks for them.

---

## Glossary

- **ASDD** — Agentic Spec-Driven Development, this methodology.
- **AC** — Acceptance Criteria; each written as `Given / When / Then`.
- **DoR** — Definition of Ready; the Phase 0 gate.
- **DoD** — Definition of Done; the exit criteria for each phase.
- **PR** — Pull Request; opened against the base branch declared in `AGENTS.md`.
- **MCP** — Model Context Protocol; the mechanism for exposing external tools
  (ticketing, design, docs) to the agent.
- **Slice** — A vertical, independently shippable unit of work; one slice =
  one ticket = one PR.
- **Gate** — A checkpoint (human or automated) that blocks progression until it
  passes.
- **Loop** — The self-verification cycle applied to every AI artifact.
- **Convergence** — Loop exit when a full review pass yields zero new
  improvements.
- **Agent / Subagent** — Primary agent vs. isolated agent instance used for
  reviews; the reviewer must be a fresh subagent, not the author.
- **Skill** — A reusable, phase-specific instruction file under `skills/`.
  Ships as a neutral base plus per-profile overlays.
- **Overlay** — A `<skill>.<profile>.md` file loaded on top of the base skill
  to add profile-specific inputs, steps, rubric rows, and templates.
- **Profile** — `fe` (Frontend) or `be` (Backend). Determines overlay selection
  and ticket template.
- **Rubric** — The scorecard the reviewer subagent uses; each phase overrides
  the default and the overlay refines it.
- **Touchpoint** — A concrete file/module/route/table/component impacted by the
  change.
- **Edge case** — A state that must be handled explicitly (empty, not-found,
  unauthorized, validation error, conflict, partial failure, idempotent retry,
  concurrent modification, loading, offline, soft-delete — profile-dependent).
- **GREEN / AMBER / RED** — Viability rating from Phase 1.
- **MAX_ITERATIONS** — Loop budget cap (default 5); reached ⇒ escalate.
- **Given / When / Then** — Canonical form for each AC line.
- **Dev** — The human developer who owns the human gates and runs all console
  commands.
- **Contract** — The externally visible interface of a change (BE: HTTP route,
  method, DTOs, status codes, error envelope, events; FE: component API,
  route/URL, page contract, telemetry events).
- **SSOT** — Single Source of Truth. Every rule / template / contract has
  exactly one canonical location; everything else links to it. See
  Principle #9.
- **Landmine** — A repo-wide rule that a change can silently break (auth policy,
  layering, state boundaries, error envelope, styling tokens, etc.).
- **FSD** — Feature-Sliced Design; an optional frontend architectural pattern
  (see [Optional architectural patterns](#optional-architectural-patterns)).
