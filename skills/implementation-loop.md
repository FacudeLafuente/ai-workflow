# Skill: implementation-loop

> Phase 5 — **Implementation per ticket**
>
> **Base skill** (profile-neutral). Load together with the profile overlay:
>
> - Frontend: `implementation-loop.fe.md`
> - Backend:  `implementation-loop.be.md`

## Purpose

Implement **one ticket at a time** end-to-end, driving the self-verification
loop until convergence, honoring every convention declared in the consumer
repo's `AGENTS.md`.

## When to use

After the ticket is `APPROVE`-d by the human governance gate (Phase 4). Never
implement an un-approved ticket. Tickets without dependencies may be run in
parallel by separate agent instances.

## Inputs

- The approved ticket (the four immutable sections).
- The Readiness Report (Phase 0) — for AC + states/edges.
- The Viability Report (Phase 1) — for touchpoint paths and gap list.
- The consumer repo's `AGENTS.md` (layering, HTTP boundary / persistence
  boundary, state, errors, auth, testing harness).
- Read/write access to the repo. **Read-only** access to CI (pipelines) —
  never modify CI from an implementation ticket unless the ticket explicitly
  demands it.

## Steps (base)

1. **Re-scan touchpoints before writing.** Even after Phases 1–2, the repo
   may have moved. Confirm every file path in the ticket exists (or is
   genuinely new) before opening the editor.
2. **Implement in the layer order declared by the profile overlay** (BE:
   entity → repo → service → DTO → mapper → controller → tests; FE:
   route/types → store → repository/data-access → composable/hook →
   component/view → tests). Never leak layers.
3. **Never suppress warnings, tests, or errors to make the build green.** Fix
   the root cause. If a warning is a symptom of an unresolved design
   decision, escalate rather than suppress.
4. **Never run commands.** Do not invoke build/test/dev/migration/`git`
   commands. Instead, in the delivery summary list the exact commands the
   Developer will run.
5. **Cover every AC line and every non-`N/A` state** from the Phase 0 matrix
   with implementation + tests.
6. **Log iterations.** Keep an `Iteration log` with one entry per loop cycle:
   what the reviewer subagent flagged, which root cause was fixed, which
   file changed.
7. **Handoff.** When the loop converges, emit the Implementation Handoff
   Report (base template below). On budget exhaustion, emit the *Unresolved &
   Escalations* section and stop — do not mark the ticket done.

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                          | Pass criterion                                                                                                     |
|---|--------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 1 | AC coverage in code and tests  | Every AC line in the ticket has (a) implementation and (b) at least one test asserting the observable outcome.     |
| 2 | Edge-case coverage             | Every non-`N/A` state from the Phase 0 matrix has a code path and a test (or an explicit `N/A — <reason>`).        |
| 3 | Layering / boundaries          | Layers respected per AGENTS.md; no cross-layer leaks.                                                              |
| 4 | Error discipline               | Errors flow through the repo's declared handling (uniform envelope / typed errors / global handler / UX).          |
| 5 | Auth / guards                  | Auth declaration present on every non-public endpoint / gated route. No relaxation of an existing policy.           |
| 6 | Contract / design discipline   | Profile-specific: contract fidelity for BE, design fidelity for FE (per overlay rubric).                            |
| 7 | Root-cause fixes only          | No warnings suppressed, no tests deleted or narrowed, no scope trimmed to dodge a failing check.                    |
| 8 | Handoff completeness           | Iteration log + list of exact commands for the Dev + Unresolved section (if any) are present.                       |

## DoD / Exit

- Loop converged (rubric all-green + zero new findings on a full pass), **or**
- Loop hit budget → *Unresolved & Escalations* emitted and the ticket is
  **not** marked done.
- Implementation Handoff Report ready for Phase 6 (human review & testing).

### Implementation Handoff Report template (base)

````md
# Implementation Handoff — Ticket #<ID>

- **Profile:** fe | be
- **Result:** Converged | Budget-exhausted (see Unresolved below)
- **Iterations:** N / MAX_ITERATIONS

## Files changed

- `<path>` — <what>
- `<path>` — <what>

## Iteration log

- **it. 1:** reviewer flagged <finding> → <root-cause fix>.
- **it. 2:** reviewer flagged <finding> → <root-cause fix>.
- **it. 3:** full pass, zero new findings → Convergence.

## Manual commands for the Developer (AI never runs these)

```bash
<install / restore>
<build>
<test>
<migration if applicable>
```

## Unresolved & Escalations

<omit this section on Convergence; required on Budget>

- **What failed:** …
- **Attempted fixes:** …
- **Root cause:** …
- **Decision required:** …
- **Blocking status:** blocks ticket #<ID>.
````

## Tools / MCP hooks

- Repo read/write.
- Subagent orchestration for the reviewer step of the loop.
- **No** command execution.
