# Skill: ticket-quality-gate

> Phase 4 — **Human Ticket Review & Governance** (human-owned gate,
> AI-assisted)
>
> **Base skill** (profile-neutral). Load together with the profile overlay:
>
> - Frontend: `ticket-quality-gate.fe.md`
> - Backend:  `ticket-quality-gate.be.md`

## Purpose

Assist the human reviewer at the governance gate: verify each ticket meets
the ASDD quality bar **before** any code is written. This skill produces a
Governance Report; approval or rejection remains a human decision.

## When to use

Immediately after `ticket-author` emits (or links) all tickets for the feature.
Never skip: this is the last cheap point at which mis-slicing or missing
contracts/designs can be fixed.

## Inputs

- The set of tickets emitted / linked in Phase 3.
- The Action Plan (Phase 2) as the source of truth for slicing and dependencies.
- The Readiness Report (Phase 0) as the source of truth for AC traceability.
- The consumer repo's `AGENTS.md` for the repo-wide rules.
- The profile ticket template for format checks.

## Steps (base)

For each ticket, run the checklist below and produce a Governance Report row.

1. **Format exactness.** Confirm the four immutable headings are present, in
   order, and not renamed. Names per profile:
   - FE: `ToDo`, `AC`, `Links to Figma`, `Notes & Obvs`.
   - BE: `ToDo`, `AC`, `Contracts & Specs`, `Notes & Obvs`.
   Reject on any deviation.
2. **Mixed-repo profile declared.** If the consumer repo declares
   `asdd_profile: mixed` in its root `AGENTS.md`, confirm the ticket's
   effective profile is resolvable — either the ticket carries a YAML
   frontmatter block declaring `profile: fe` (or `be`) before the title,
   or every touchpoint of the ticket falls under a folder covered by a
   per-package `AGENTS.md` or a `asdd/config.yml` `packages:` entry.
   Reject if the effective profile cannot be resolved. Skip this check
   for repos declared `asdd_profile: fe` or `asdd_profile: be`.
3. **Granularity.** The ticket represents exactly one slice from the plan. If
   the diff would touch two unrelated behaviors, reject → back to Phase 3
   (split).
4. **PR-sized.** The described work fits a single focused PR (see the Phase 2
   sizing rules). Reject on obvious over-scope.
5. **Parallelizable.** Dependencies are explicit as `blocked by #<ID>` /
   `blocks #<ID>`. Reject on hidden dependencies.
6. **Governable.** One stage of work — no "and then also do X for the next
   feature" trailers.
7. **Traceable.** Every AC line traces to a Business/Product criterion from
   the Readiness Report. Reject on invented ACs.
8. **Repo conventions honored** (per profile overlay).
9. **Self-contained.** An implementer can build the slice from this ticket
   alone, without opening chat history, meeting notes, or a design doc.
10. **Profile-specific landmines checked** (per overlay).
11. **Emit the Governance Report** (base template below) and hand it to the
    human reviewer. Approval / rejection is a **human decision** — the skill
    only surfaces evidence.

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| 1 | Format exactness             | Four headings present, in order, exact names per profile.                                                   |
| 2 | Granularity + PR-sizing      | Single slice; fits a focused PR.                                                                            |
| 3 | Dependency explicitness      | `blocks` / `blocked by` reflect the plan; no hidden order.                                                  |
| 4 | Traceability                 | Every AC line links to a Business/Product AC.                                                               |
| 5 | Repo conventions honored     | AGENTS.md-declared rules are all explicit in the ticket.                                                    |
| 6 | Landmine sweep               | Profile-specific landmines resolved or flagged as reject reasons.                                           |
| 7 | Human-actionable report      | The report tells the human exactly what to APPROVE, REJECT (with reason), or ASK-FOR-CHANGES.               |

## DoD / Exit

- Governance Report emitted with a row per ticket and a single verdict
  (`APPROVE` / `REQUEST CHANGES` / `REJECT — split/refit`).
- Human reviewer signs off on the tickets.
- Approved → Phase 5 (`implementation-loop`) may start.
- Changes requested → back to Phase 3 (`ticket-author`).

### Governance Report template (base)

```md
# Governance Report — <feature name>

- **Profile:** fe | be

| # | Ticket | Format | Granular | PR-sized | Deps explicit | Traceable | Repo conv. | Landmines | Verdict | Notes |
|---|--------|--------|----------|----------|---------------|-----------|------------|-----------|---------|-------|
| 1 | #<ID> | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | none | APPROVE | — |
| 2 | #<ID> | ✅ | ❌ | — | — | — | — | — | REJECT — split | Slice touches two behaviors; split into #<ID>-a and #<ID>-b. |
| 3 | #<ID> | ✅ | ✅ | ✅ | ✅ | ✅ | ⚠️ | schema | REQUEST CHANGES | Missing migrations strategy; blocked pending flow decision. |

## Human decision required

- Ticket #<ID>: <exact decision the human must make>.
```

## Tools / MCP hooks

- Read-only access to the ticket bodies.
- Diff against the profile ticket template to detect format drift.
