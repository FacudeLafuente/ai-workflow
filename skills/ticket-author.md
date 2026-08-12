# Skill: ticket-author

> Phase 3 — **Ticketing**
>
> **Base skill** (profile-neutral). Load together with the profile overlay:
>
> - Frontend: `ticket-author.fe.md` (uses `templates/ticket.fe.md`)
> - Backend:  `ticket-author.be.md` (uses `templates/ticket.be.md`)

## Purpose

Turn each slice from the Action Plan into a **Work Item** using the immutable
format from `templates/ticket.<profile>.md`. One slice ⇒ one ticket ⇒ one PR.

## When to use

After `action-plan-builder` produces an approved Action Plan.

- **Mode A** — emit brand-new tickets from the plan.
- **Mode B** — the ticket already exists; only validate/link it against the
  template. Split only if the plan requires.

## Inputs

- Action Plan (Phase 2) with slice list, dependency graph, AC coverage matrix.
- Readiness Report (Phase 0) for the AC source text and the states/edges matrix.
- Viability Report (Phase 1) for the concrete file touchpoints.
- The active profile's ticket template (`templates/ticket.<profile>.md`).
- The consumer repo's `AGENTS.md` (ticket ID format, ticketing system, base
  branch, layering / conventions).

## Immutable format

Every ticket has four headings, in this exact order, per profile:

| # | Heading (FE profile) | Heading (BE profile) |
|---|----------------------|----------------------|
| 1 | `ToDo`               | `ToDo`               |
| 2 | `AC`                 | `AC`                 |
| 3 | `Links to Figma`     | `Contracts & Specs`  |
| 4 | `Notes & Obvs`       | `Notes & Obvs`       |

Empty section ⇒ `N/A`. Do not add extra top-level headings. Do not rename.

## Steps (base)

1. **One slice → one ticket.** Never merge two slices into one ticket, and
   never split one slice across two tickets (that means the slice was
   mis-sized — return to Phase 2).
2. **Fill the four immutable sections** from the profile template, in order.
3. **ToDo — atomic actions.** Each item must be verifiable from the diff.
   The profile overlay lists the discipline items that must appear whenever
   applicable (auth annotation, response-type metadata, mapping, test class,
   accessibility, telemetry, feature flag).
4. **AC — copy the normalized Given/When/Then** from the Readiness Report
   for this slice's AC subset. Always include the explicit "Edge cases /
   states covered" line, marking inapplicable states as `N/A — <reason>` —
   never silently omit.
5. **Heading #3 — profile-specific.**
   - BE: fill the `Contracts & Specs` section per the profile overlay
     (endpoint table, request/response DTOs, error envelope, persistence,
     integrations, feature flag, NFRs).
   - FE: fill the `Links to Figma` section per the profile overlay (screens,
     component library refs, design system references, state variants).
6. **Notes & Obvs — repo-specific context.**
   - Touchpoints (file paths, one per line).
   - Dependencies as `blocked by #<ID>` / `blocks #<ID>`.
   - Governance / risk (auth, data exposure, backward compatibility, breaking
     UX / API change).
   - Out-of-scope (explicit list — otherwise reviewers will assume the slice
     covers it).
   - Manual commands the Dev will run (ASDD forbids the AI from running them).
7. **Cross-ticket linking.** Every dependency listed in the plan must appear
   as `blocked by #<ID>` / `blocks #<ID>`. If the blocker's ID is not yet
   known, emit tickets in dependency order and back-fill IDs before Phase 4.
8. **Mode B validation.** For each existing ticket:
   - Confirm the four immutable headings exist, in order, with the exact names
     for the active profile.
   - Confirm the ticket represents a single slice (matches one row of the plan).
   - If the ticket is oversized (multiple slices), split into PR-sized tickets;
     otherwise only validate and link.

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                        | Pass criterion                                                                                                       |
|---|------------------------------|----------------------------------------------------------------------------------------------------------------------|
| 1 | Format exactness             | Four headings present, in order, with the exact names from the profile template. Empty sections ⇒ `N/A`.             |
| 2 | Granularity                  | One slice, one ticket. No merged/split slices.                                                                       |
| 3 | Traceability                 | Every AC line in the ticket maps back to a Business/Product AC in the Readiness Report.                              |
| 4 | Heading-3 completeness       | The profile-specific 3rd section (Contracts & Specs or Links to Figma) is complete per the overlay's rubric.         |
| 5 | Repo conventions             | Auth, layering, error handling, styling / naming, testing — all explicitly addressed per AGENTS.md.                  |
| 6 | Dependencies wired           | `blocks` / `blocked by` correctly reflect the plan's dependency graph.                                               |
| 7 | Self-contained               | An implementer can build the slice from the ticket alone (no external context required).                             |

## DoD / Exit

- All slices materialized as Work Items in the immutable format.
- Cross-ticket links (`blocked by` / `blocks`) resolved.
- Hand off to Phase 4 (`ticket-quality-gate`) for the human governance review.

## Tools / MCP hooks

- Ticket templates: `templates/ticket.fe.md`, `templates/ticket.be.md`.
- Ticketing MCP (roadmap) — create/link Work Items / Issues directly from the
  emitted Markdown.
