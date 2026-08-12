# Prompt: Self-verification loop

> Reusable engine for **every** AI artifact produced under ASDD (see
> [`../ASDD.md`](../ASDD.md)).
>
> **Draft → Review (fresh subagent, evaluates ONLY against rubric + AC) → Fix
> root cause → Repeat.**

This file has two parts. The **formal specification** (sections 1–7) defines
the contract, parameters, pseudocode, default rubric, anti-patterns, exit
conditions, and the *Unresolved & Escalations* template. The **drop-in prompt
block** (section 8) is a ready-to-paste text you can hand to any agent.

---

## 1. Contract

- The **Author** produces a draft artifact (spec, plan, ticket, code diff, PR
  description, …).
- The **Reviewer** is a **fresh subagent instance** that has never seen the
  draft before. It evaluates the draft **only** against the rubric + the ACs
  it was given. It never rewrites the artifact.
- The **Author** applies fixes — always addressing the **root cause**, never
  silencing or narrowing scope.

**Reviewer bias is the #1 anti-pattern.** The author must not re-read its own
draft and mark it "good enough". Always spawn a fresh subagent for the review
step, or explicitly switch role and forget prior context.

---

## 2. Loop parameters

| Parameter        | Default                  | Meaning                                                                 |
|------------------|--------------------------|-------------------------------------------------------------------------|
| `MAX_ITERATIONS` | `5`                      | Hard budget cap for `draft → review → fix` cycles.                      |
| `PHASE_RUBRIC`   | —                        | Rubric table used by the reviewer (each phase overrides the default).   |
| `AC`             | —                        | The Acceptance Criteria from Phase 0 (normalized as `Given/When/Then`). |
| `PROFILE`        | `fe` or `be`             | Active ASDD profile; determines which overlay rubric rows apply.        |
| `STOP_ON`        | Convergence **or** Budget | Only two legitimate exit conditions.                                   |

---

## 3. Loop pseudocode

```text
iteration = 0
draft     = <initial artifact>

while iteration < MAX_ITERATIONS:
    iteration += 1

    review = fresh_subagent.evaluate(
        artifact = draft,
        rubric   = PHASE_RUBRIC,   # merged: default + phase override + profile overlay
        ac       = AC,
        profile  = PROFILE,
    )
    # review = { failures[], warnings[], suggestions[], full_pass_no_improvements: bool }

    if review.failures.empty and review.full_pass_no_improvements:
        return Convergence(draft)

    draft = author.fix_root_causes(
        artifact = draft,
        findings = review,
    )

return Budget(draft, review)  # escalate: MUST include "Unresolved & Escalations"
```

---

## 4. Default rubric (each phase overrides; overlay refines)

| # | Check                        | Pass criterion                                                                                                                                     |
|---|------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 1 | AC coverage                  | Every AC line is verifiably addressed by the artifact. No orphan, no duplicate.                                                                    |
| 2 | Scope fidelity               | Nothing outside the declared scope was added. Nothing declared in scope is missing.                                                                |
| 3 | Root-cause fixes             | No workaround silences a failing check (warning ignored, test deleted, scope trimmed, feature flag hiding a bug).                                  |
| 4 | Repo conventions             | Honors the rules declared in the consumer repo's `AGENTS.md` (layering, auth, error handling, state management, styling, testing harness).         |
| 5 | States & edges               | Every state/edge from the Phase 0 matrix is either implemented or marked `N/A — <reason>` (never silently omitted).                                |
| 6 | Consistency                  | Names, paths, types, and status codes are consistent between the artifact and every other artifact of the same ticket (spec ↔ plan ↔ ticket ↔ code ↔ PR). |
| 7 | Self-contained handoff       | The next owner (human or subagent) can act on the artifact **without** external context.                                                           |
| 8 | SSOT compliance              | The artifact does not restate a rule, contract, or content that already lives in another canonical file. It **links** to the source. Duplication is a defect. See [`ASDD.md` → Principles](../ASDD.md#principles) (Principle #9). |
| 9 | Comment discipline           | Generated code carries **no inline comments** except (a) genuinely necessary, (b) important non-obvious context, or (c) explicitly requested by the user. The only agent-initiated comment allowed is a **file-header of ≤3 lines**. No commented-out code, no `TODO`/`FIXME`, no docstrings on internal / private functions unless a public API demands them. See [`AGENTS.md` → Rule #6](../AGENTS.md). |

Profile overlays add rows such as **contract discipline** (BE — HTTP method +
route + policy + error envelope) or **design fidelity** (FE — Figma link +
component library reference + design tokens + accessibility).

---

## 5. Anti-patterns (auto-fail the review)

- **Infinite loops with no convergence signal.** If the reviewer keeps
  producing new findings on identical passes, the loop is diverging — stop,
  escalate.
- **Green by deletion.** Passing checks by removing the failing test, the
  failing AC, or the failing edge case. If a check cannot be met, escalate.
- **Hidden debt.** Marking a ticket done while leaving TODOs, commented-out
  code, or unresolved warnings that pertain to the ticket's scope.
- **Reviewer bias.** The reviewer must be a fresh subagent. A re-read by the
  author is not a review.
- **Scope drift.** Adding refactors, "improvements", or defensive layers that
  were not in the ticket. Propose them as a follow-up ticket instead.
- **Command execution.** The AI never runs build/test/dev/migration/`git`
  commands. It lists the exact commands for the Developer to run.
- **Faked command results.** Never claim the output of a command the AI did
  not (and by policy cannot) run.

---

## 6. Exit conditions

- **Convergence.** All rubric checks pass and a full review pass produces zero
  new improvements → hand off.
- **Budget.** `MAX_ITERATIONS` reached without convergence → **do not** hand
  off as complete. Emit an *Unresolved & Escalations* section listing:
  - What is still failing (which rubric row, which AC).
  - Why the root cause could not be resolved in-loop (missing information,
    blocked by a decision, requires a schema / pipeline / design change).
  - The exact decision required from the human.

---

## 7. `Unresolved & Escalations` section (required on Budget exit)

```md
## Unresolved & Escalations

- **What failed:** <rubric row / AC line>.
- **Attempted fixes:** <iteration 1 → …, iteration N → …>.
- **Root cause:** <one-sentence diagnosis>.
- **Decision required:** <exact question for the human>.
- **Blocking status:** <blocks ticket #, or non-blocking follow-up>.
```

---

## 8. Drop-in prompt block (paste to any agent)

Copy the fenced block below after your task-specific instructions so the agent
(and its subagents) iterate on their own output, attacking the **root cause**
of every defect, until the work can no longer be improved — and only then
hand off to the human.

> Output language: the language of the requesting user (default: English for
> code artifacts; prose can follow the user's language).

```text
You will not deliver on the first attempt. Run a self-verification loop:

1. DRAFT — Produce the artifact requested above.

2. REVIEW — Spawn a fresh reviewer subagent with NO memory of your reasoning.
   It must evaluate the draft ONLY against the rubric/checklist below and the
   provided Acceptance Criteria. It returns: PASS/FAIL per item + concrete
   defects.

3. FIX — For every FAIL, fix the ROOT CAUSE. Forbidden: silencing warnings,
   narrowing scope to dodge a check, try/catch that hides errors, TODOs left
   as "done", or any workaround that masks the problem. If a defect cannot be
   fixed, record it explicitly (do not hide it).

4. REPEAT — Go back to step 2 with the revised artifact.

STOP when EITHER:
  (a) CONVERGENCE — every checklist item is PASS AND a full review pass
      produced zero new actionable improvements; OR
  (b) BUDGET — you reached {{MAX_ITERATIONS}} iterations (default 5).

HANDOFF — Output:
  - The final artifact.
  - An iteration log (what changed each pass, why).
  - A "Commands to run (manual)" list when the task implies build/test/dev/git:
    NEVER run console commands yourself — list the exact commands for the
    human and never claim a command result you did not actually obtain.
  - If stopped by (b): an "Unresolved & Escalations" section listing every
    open item, the root cause hypothesis, and what you tried. NEVER pretend
    it is done.
```
