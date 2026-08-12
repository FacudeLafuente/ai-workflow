# Skill: spec-intake

> Phase 0 — **Intake Gate (Definition of Ready)**
>
> **Base skill** (profile-neutral). Always load together with the profile
> overlay for the active repo:
>
> - Frontend: `spec-intake.fe.md`
> - Backend:  `spec-intake.be.md`

## Purpose

Turn business input into a **Readiness Report** that either unblocks the flow
(`READY`) or halts it (`BLOCKED`). The output is the immutable spec every
downstream phase will trust.

## When to use

- **Mode A** — the request arrives as raw business context (a Slack thread, a
  meeting note, a ticket comment, a stakeholder ask).
- **Mode B** — the request arrives as an already-groomed ticket. The skill
  **validates** the ticket against the DoR checklist; it does not rewrite it.

## Inputs (base — profile-agnostic)

- Business / Product Acceptance Criteria (any form).
- Use cases, states, needs.
- **Non-functional needs** — auth policy / guard, performance, pagination,
  idempotency, concurrency, PII / exposure, feature flag key.

Profile overlays add:

- Contract expectations (BE: HTTP routes + DTOs + status codes + error
  envelope mapping; FE: screens + Figma nodes + component library refs +
  design tokens).
- Data touched (entities / tables / stores).
- External integrations, events, background jobs.

## Steps (base)

1. **Detect entry mode** (A: raw context, B: groomed ticket) and map every
   required item to `present / missing / ambiguous`.
2. **Collect** every input above. Do not invent. If a value is missing, log it
   in the DoR checklist as `MISSING — <what is needed> — <who to ask>`.
3. **Normalize AC** to `Given / When / Then`, one line per outcome. Keep the
   business phrasing where possible.
4. **Build the states/edge-cases matrix.** For every AC line, decide whether
   each of the following applies: empty, not-found, unauthorized, forbidden,
   validation error, conflict, rate-limit, partial failure, idempotent retry,
   concurrent modification. Mark inapplicable states as `N/A — <one-sentence
   reason>` (never silently omit). The profile overlay adds profile-specific
   states (loading / offline / soft-delete for FE; upstream unavailable /
   schema-change for BE).
5. **Apply the profile overlay's contract sanity-check.**
6. **DoR checklist** (all must be `✓` to emit `READY`) — see the profile
   overlay for the full row list.
7. **Emit the Readiness Report** (base template below; the overlay may add
   sections).
8. **Mode B extra step.** Cross-check the existing ticket against the profile
   template (`templates/ticket.<profile>.md`). If any of the four sections is
   missing or renamed, flag it — do not silently accept.

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the **default rubric** plus the phase-specific rubric override below and the
profile overlay's rubric additions.

**Phase override (base):**

| # | Check                        | Pass criterion                                                                                                        |
|---|------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 1 | AC completeness              | Every stakeholder need is covered by a `Given/When/Then` line; no orphan need, no duplicate AC.                       |
| 2 | Edge-case coverage           | Matrix explicitly resolves every candidate state (implement or `N/A — <reason>`).                                     |
| 3 | Auth explicitness            | Every endpoint / route / action has a declared policy / guard, or a public-endpoint justification.                    |
| 4 | No invention                 | No value is assumed. Missing inputs appear in the DoR checklist as `MISSING — …`.                                     |
| 5 | Handoff to Phase 1           | A Phase 1 agent can execute `repo-viability-scan` from this report alone (no missing links).                          |

## DoD / Exit

- Emit a **Readiness Report** with header `# Readiness Report — <feature name>`.
- Verdict is exactly one of `READY` or `BLOCKED`.
- On `READY`: attach the normalized AC list + states/edge-cases matrix +
  contract/design block + non-functional block. Hand off to Phase 1
  (`repo-viability-scan`).
- On `BLOCKED`: attach the DoR checklist with every `MISSING — …` entry and
  the named owner to unblock. Do **not** proceed.

### Readiness Report template (base)

```md
# Readiness Report — <feature name>

- **Profile:** fe | be
- **Mode:** A (raw context) | B (groomed ticket #<ID>)
- **Verdict:** READY | BLOCKED
- **Source ticket / thread:** <link>

## Acceptance Criteria (normalized)

- Given … When … Then …
- Given … When … Then …

## States / edge cases

| State | Applies? | Expected behavior |
|-------|----------|-------------------|
| empty | yes/no | … |
| not-found | yes/no | … |
| unauthorized | yes/no | … |
| forbidden | yes/no | … |
| validation error | yes/no | … |
| conflict | yes/no | … |
| rate-limit | yes/no | … |
| partial failure | yes/no | … |
| idempotent retry | yes/no | … |
| concurrent modification | yes/no | … |
| <profile-specific state> | yes/no | … |

## Non-functional

- Auth policy / guard: `<name>` (or `public — <reason>`).
- Idempotency: …
- Pagination: …
- PII / exposure: …
- Feature flag: … (default per env)

## DoR checklist

- [x] Business ACs present and unambiguous.
- [ ] MISSING — <what> — <who to ask>
```

The profile overlay adds an "Acceptance Criteria — contract" section (BE) or
an "Acceptance Criteria — design & UX" section (FE) to the template.

## Tools / MCP hooks

- Repo introspection (file / text search).
- Profile overlay adds MCP hooks (e.g. Figma MCP for FE, OpenAPI MCP for BE).
