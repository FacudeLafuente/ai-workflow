# Skill: action-plan-builder

> Phase 2 — **Planning (Action Plan)**
>
> **Base skill** (profile-neutral). Load together with the profile overlay:
>
> - Frontend: `action-plan-builder.fe.md`
> - Backend:  `action-plan-builder.be.md`

## Purpose

Turn the Readiness Report + Viability Report into an **Action Plan** made of
**vertical slices**, each independently shippable as a small, focused PR.

## When to use

After `repo-viability-scan` returns `GREEN` or `AMBER`. Do not plan on `RED`.

## Inputs

- The Readiness Report (Phase 0).
- The Viability Report (Phase 1) including the gap list.
- The repo's `AGENTS.md` (layering, module boundaries, base branch, ticket ID
  format).
- The ticket template for the active profile (`templates/ticket.<profile>.md`).

## Steps (base)

1. **Draft candidate slices.** Prefer **vertical cuts** (one behavior
   end-to-end across all layers of the profile) over horizontal cuts (all
   controllers first, then all services — FE equivalent: all components, then
   all stores). Horizontal slices almost always violate the "one shippable
   slice" principle.
2. **Right-size each slice to a small PR.** Rule of thumb:
   - ≤ ~300 lines of diff (excluding generated code / migrations).
   - Touches one behavior / one endpoint / one screen.
   - Fits one review session end-to-end.
   - If bigger, split it — the overlay lists the typical split axes per
     profile.
3. **Per slice, fill in:**
   - Objective (one sentence).
   - AC subset from the Readiness Report (map by ID/line).
   - Touchpoints from the Viability Report (paths).
   - States/edges from the Phase 0 matrix that this slice must cover.
   - Risks (auth regression, breaking contract, data migration, feature flag,
     performance, accessibility, breaking UX).
   - Dependencies: `blocks` / `blocked by`.
   - Orchestration hint: which agent/subagent/skill/MCP is expected in Phase
     5, and whether a loop is required (default: yes).
4. **Build the dependency graph.**
   - Must be **acyclic**. If a cycle appears, re-slice.
   - Mark parallelizable slices (`||`) explicitly so multiple developers /
     agents can pick them up.
   - Slices that only add a helper with no consumer are usually a smell —
     merge them into their consumer slice or justify why they ship alone.
5. **AC coverage check.**
   - Every AC line is covered by exactly one slice (no orphans, no duplicates).
   - If an AC needs contributions from multiple slices, split it into
     sub-criteria and reassign so each sub-criterion belongs to one slice.
6. **Landmine-aware ordering.**
   - Schema-change / cross-cutting slices (new error mapping, new auth
     policy, new design-system token) ship before any slice that depends on
     them.
   - The overlay lists profile-typical cross-cutting slices.
7. **Emit the Action Plan** (base template below; overlay may add sections)
   and hand off to Phase 3 (`ticket-author`).

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| 1 | AC coverage                  | Every AC line maps to exactly one slice; no orphan, no duplicate.                                           |
| 2 | Verticality                  | Each slice covers a full stack of the profile (or explains why not).                                        |
| 3 | PR-sizing                    | No slice is too big for a single focused PR; oversized slices are split.                                    |
| 4 | Dependency graph             | Graph is acyclic; blockers precede blocked slices; parallelizable slices are marked.                        |
| 5 | Landmine ordering            | Schema-change / cross-cutting slices are correctly ordered (or blocked pending human decision).             |
| 6 | Orchestration hints          | Each slice names the intended agent/subagent/skill/MCP and whether a loop is required.                      |
| 7 | Handoff to Phase 3           | `ticket-author` can emit tickets directly from this plan without re-reading Phases 0–1.                     |

## DoD / Exit

- Action Plan emitted with the slice table + dependency graph + AC coverage
  matrix.
- All slices sized for a single PR; graph acyclic; AC coverage complete.
- Hand off to `ticket-author`.

### Action Plan template (base)

````md
# Action Plan — <feature name>

- **Profile:** fe | be
- **Rating carried over:** GREEN | AMBER (+ open questions)
- **Total slices:** N (M parallelizable)

## Slices

### S1 — <short imperative title>

- **Objective:** …
- **AC covered:** AC-1, AC-3 (from Readiness Report).
- **Touchpoints:**
  - `<path 1>`
  - `<path 2>`
- **States/edges:** empty, error, forbidden, validation.
- **Risks:** <auth regression / breaking contract / UX regression / …>.
- **Dependencies:** blocked by S0; blocks S2, S3.
- **Orchestration:** `implementation-loop` skill, loop required (default budget).
- **PR size estimate:** small (~150–250 lines).

### S2 — …

## Dependency graph

```mermaid
graph LR
  S0[S0 schema/policy] --> S1
  S0 --> S2
  S1 --> S3
  S2 --> S3
  S4[S4 || parallel] -.-> S3
```

## AC coverage matrix

| AC line | Covered by |
|---------|------------|
| AC-1 | S1 |
| AC-2 | S2 |
| AC-3 | S1 |
| AC-4 | S3 |

## Open questions carried

- [AMBER from Phase 1] <question> — owner: <who> — required before S<slice>.
````

## Tools / MCP hooks

- Repository read (for touchpoint validation).
- Mermaid rendering for the dependency graph.
- Ticketing MCP (roadmap) — draft Work Items / Issues directly from each slice.
