---
# ASDD base AGENTS.md — canonical specification of what any consumer
# repo's AGENTS.md must contain to work with ASDD.
#
# This file is BOTH:
#   (a) The single source of truth for the AGENTS.md shape and its shared
#       rules (workflow discipline, ticket format, no-commands, root-cause).
#       Any change here MUST be reflected in `templates/AGENTS.md.fe.example`
#       and `templates/AGENTS.md.be.example`, and vice versa.
#   (b) A living spec — reading this file gives a new consumer or agent the
#       full picture of what an AGENTS.md-driven repo looks like.
#
# It is NOT copied verbatim to the consumer repo. The consumer copies one of
# the profile-specific templates from `templates/AGENTS.md.<profile>.example`,
# which already inline the base + the profile deltas.
---

# AGENTS.md — canonical base (ASDD-compatible)

> **Read this file first if you are authoring or reviewing an `AGENTS.md`
> for a repo that uses ASDD.** For a ready-to-copy starting point, use
> [`templates/AGENTS.md.fe.example`](templates/AGENTS.md.fe.example),
> [`templates/AGENTS.md.be.example`](templates/AGENTS.md.be.example),
> or [`templates/AGENTS.md.mixed.example`](templates/AGENTS.md.mixed.example)
> if the same repo hosts both FE and BE code.

## Purpose

`AGENTS.md` is the **canonical source of code conventions** for a repository
that adopts ASDD. It answers the question *"how does work get done here?"*
so an AI agent (or a new developer) can plan, implement, test, commit and
ship without guessing.

- ASDD (in [`ASDD.md`](ASDD.md)) owns the **workflow** — how work moves
  from a business idea to a merged PR.
- `AGENTS.md` in the consumer repo owns the **conventions** — how the code
  looks, what layers exist, which commands run, what NOT to do.

The two never overlap. Rules about the workflow live in `ASDD.md`; rules
about the code live in the consumer's `AGENTS.md`. Neither duplicates the
other — **Single Source of Truth** is a first-class principle (see
[`ASDD.md` → Principles](ASDD.md#principles)).

## Canonical structure — 9 sections, in this order

Every `AGENTS.md` derived from this base follows exactly this shape:

```text
# AGENTS.md

## Project Overview          ← what it is, stack, purpose (1 paragraph)
## Project Layout            ← only NON-obvious bits, not a folder tree
## Domain Core               ← business concepts the code does not explain
## Tooling                   ← MCPs, skills, external services preferred
## Basic Project Commands    ← install, dev, build, test, lint (verified)
## Testing                   ← runner, harness, where tests live
## Code Conventions          ← style, patterns, imports, state, errors
## Commit & PR Conventions   ← commit format, CI checks, branch naming
## What NOT to Do            ← explicit prohibitions
```

- **Do not** add extra top-level headings. Anything that does not fit above
  goes as a subsection (`###`) of the closest match.
- **Do not** rename. Format-exactness lets the agent index sections
  reliably across every ASDD-compatible repo.
- Empty section ⇒ write a one-line `N/A — <reason>`. Never silently omit.

The three profile templates (`AGENTS.md.fe.example` /
`AGENTS.md.be.example` / `AGENTS.md.mixed.example`) implement this shape
with realistic starting content per stack. The BE template uses
subsections inside `## Code Conventions` (Layering & DTO placement / API
conventions / Error envelope / Persistence & migrations / Style) to keep
the 9 top-level sections stable. The mixed template splits
`## Code Conventions` into `### Frontend` and `### Backend` subsections
and points at the FE / BE templates for the concrete rules — it never
duplicates them (SSOT).

## Required frontmatter — ASDD profile & wiring

Every consumer `AGENTS.md` starts with this YAML frontmatter block so ASDD
knows the profile and delivery parameters:

```yaml
---
asdd_profile: fe          # or "be", or "mixed" (FE + BE in the same repo)
base_branch: development  # branch pr-handoff will target
ticketing:
  system: azure-devops    # or: github-issues, jira, linear, gitlab
  ticket_id_pattern: "#<NUMBER>"
---
```

Allowed values for `asdd_profile:`:

- `fe` — Frontend-only repo. All skills load their `.fe` overlays.
- `be` — Backend-only repo. All skills load their `.be` overlays.
- `mixed` — Same repo hosts both FE and BE code (monorepo with FE + BE
  packages, or a monolith with FE + BE intermingled). The effective
  profile is resolved **per ticket**: each ticket declares
  `profile: fe` or `profile: be` in its frontmatter, and every skill
  loads the matching overlay for that ticket. See
  [`ASDD.md` → Profiles](ASDD.md#profiles) for the resolution rules and
  the halt behavior when a ticket in a `mixed` repo does not declare a
  profile.

Precedence for profile resolution: this frontmatter → `asdd/config.yml`
→ autodetection by `repo-viability-scan`. See [`ASDD.md` → Profiles](ASDD.md#profiles).

## Shared rules that every AGENTS.md must enforce

These are the rules that come **from ASDD** and must be honored by any
consumer, regardless of profile or stack. Both profile templates already
include them; do not remove them when adapting the template.

### 1. Console commands are humans-only

- The AI **never** runs `<install> / <build> / <test> / <lint> / <migrate> /
  git *` commands. It lists the exact commands in the delivery summary and
  the Developer runs them.
- Never claim the output of a command the AI did not (and by policy cannot)
  run.

### 2. Root cause, not shortcuts

- Do not silence warnings, delete tests, narrow scope, add `try/catch` that
  hides errors, comment out failing code, or flip feature flags to make a
  check pass.
- If a defect cannot be fixed within scope, emit an *Unresolved &
  Escalations* section (per [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md)).

### 3. Ticket format is immutable

- Every ticket has four headings in exact order:
  - FE profile: `ToDo / AC / Links to Figma / Notes & Obvs`.
  - BE profile: `ToDo / AC / Contracts & Specs / Notes & Obvs`.
- Empty section ⇒ `N/A`. Do not rename, reorder, or add extra top-level
  headings. See `templates/ticket.<profile>.md`.

### 4. Single Source of Truth (SSOT)

- Do not restate a rule that already lives in another canonical file. Point
  at it.
- If two files contradict each other, the more specific one wins (skill
  overlay over base skill, `AGENTS.md` frontmatter over `config.yml`), and
  the contradiction is fixed at the source — never worked around.
- Duplicating the same rule in multiple places is a defect the reviewer
  will flag.

### 5. Delivery discipline

- One ticket → one focused PR.
- Branch: `<prefix>/<TICKET-ID>-<kebab-slug>` off the declared `base_branch`.
- Conventional commits with the ticket ID (format declared in
  `## Commit & PR Conventions`).
- No `git push --force` on shared branches. No `--no-verify`.

### 6. Minimal comments in generated code

- The agent **does not add inline comments** to code it generates or edits.
  Comments are permitted only when:
  1. **Genuinely necessary** to make the code correct or understandable
     (non-obvious algorithm, security constraint, workaround for a known
     bug, cross-file invariant).
  2. **Important context** that is not visible from the code itself
     (ticket ID, RFC link, why-not-obvious-alternative).
  3. **The user explicitly asked** for a comment.
- The **only** comment the agent may add on its own initiative is a
  **file-header block of at most 3 lines** at the top of a file it creates
  or substantially reworks (purpose, ticket reference, or agent tag).
- **Forbidden:** obvious comments (`// increment counter`), commented-out
  code, TODO/FIXME (track as follow-up tickets), docstrings for internal /
  private functions unless a public API demands them.

### 7. Token efficiency

Applies to every artifact the agent produces (code, prose, tickets, plans,
reports). The self-verification loop enforces these via its rubric.

- **Response discipline.** No preambles ("Great question", "As you can
  see", "Let me…"). No restating the user's request. No closing summaries
  after edits ("Now the code does X"). Answer format matches the ask: a
  one-line question gets a one-line answer; a confirmation gets `Done` /
  `OK`; a complex ask gets a structured response. Prefer bullets and
  tables over prose when the content is structured.
- **Tool discipline.** Batch operations: `multi_replace_string_in_file`
  over sequential `replace_string_in_file`; parallel independent reads in
  the same turn. Reference files by path / link — never quote their
  content when a link suffices.
- **Code output.** Rule #6 above (minimal comments). No obvious
  docstrings. No unnecessary logging. No defensive `try/catch` outside
  scope. Delete instead of comment-out.

### 8. Bootstrap trigger phrases

If the user says any of `Start ASDD`, `Iniciar ASDD`, `Run ASDD`, or
`asdd init` (case-insensitive), the agent must load and follow
[`prompts/bootstrap.md`](prompts/bootstrap.md) **before** answering.
The bootstrap performs a read-only detection of the connection layer at
the project root and lists the safe-copy commands the Developer must
run. The agent never runs those commands (Principle #8 in
[`ASDD.md`](ASDD.md)).

## Content rules (authoring an `AGENTS.md` that works)

The following authoring rules apply when a human or agent adapts one of the
profile templates. They are **not enforcement rules** on the runtime — they
are **quality rules** on the file itself.

- **Be specific, not generic.** ✅ *"Composition API only in new code;
  ESLint blocks Options API"* — ❌ *"write clean code"*.
- **Cite exact paths.** ✅ *"`src/composables/api/useHttpClient.ts`"* — ❌
  *"the HTTP client"*.
- **Copy-pasteable commands.** Use fenced `bash` blocks; every command must
  actually work in a fresh clone.
- **Concrete prohibitions.** ✅ *"Do not call `axios` directly"* — ❌ *"use
  the HTTP layer"*.
- **Keep it lean.** If it grows past ~300 lines, split by scope
  (subfolders can carry their own `AGENTS.md`; the closest one to the file
  being edited wins).
- **Do not duplicate the README.** The `README.md` targets humans running
  the project; `AGENTS.md` targets agents modifying it.
- **Document the "why" only when it changes decisions.** Rare constraints
  deserve a one-line rationale; obvious ones don't.

## Anti-patterns (auto-rejected by `ticket-quality-gate` and the loop reviewer)

- ❌ Reproducing the folder tree — the agent already sees it.
- ❌ Vague rules (*"write good code"*, *"follow best practices"*).
- ❌ Aspirational instructions without enforcement (*"try to use strict
  TypeScript"*).
- ❌ Duplicating content between `README.md`, `CONTRIBUTING.md`, and
  `AGENTS.md` — violates SSOT.
- ❌ Heavy YAML frontmatter beyond the ASDD block — this is not a Copilot
  `.instructions.md`.
- ❌ Style rules a formatter/linter already enforces — the tool is the
  source of truth for style.
- ❌ Unverified commands — an agent will run them and fail.
- ❌ Adding a rule here that already lives in `ASDD.md`, in a skill, or in
  a template — point at the source instead.

## Where each concern lives (map to avoid duplication)

| Concern                              | Where it lives                                              |
|--------------------------------------|-------------------------------------------------------------|
| Workflow (phases, gates, loop)       | [`ASDD.md`](ASDD.md)                                        |
| Self-verification engine             | [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md) |
| Ticket format                        | [`templates/ticket.<profile>.md`](templates/)               |
| Profile selection precedence         | [`ASDD.md` → Profiles](ASDD.md#profiles)                    |
| Base branch / ticketing system       | Consumer `AGENTS.md` frontmatter                            |
| Code layering, HTTP client, state    | Consumer `AGENTS.md` → `Code Conventions`                   |
| Auth policies / guards               | Consumer `AGENTS.md` → `Code Conventions`                   |
| Error handling / envelope            | Consumer `AGENTS.md` → `Code Conventions`                   |
| Testing harness                      | Consumer `AGENTS.md` → `Testing`                            |
| Commands (install / test / build)    | Consumer `AGENTS.md` → `Basic Project Commands`             |
| Commit format & CI checks            | Consumer `AGENTS.md` → `Commit & PR Conventions`            |
| Prohibitions                         | Consumer `AGENTS.md` → `What NOT to Do`                     |
| Optional architectural patterns (FSD, Clean, DDD, …) | [`ASDD.md` → Optional architectural patterns](ASDD.md#optional-architectural-patterns) |

If a rule fits in more than one row, put it where it belongs semantically
and **link** from the other rows — never copy.
