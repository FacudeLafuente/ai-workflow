# Skill: pr-handoff

> Phase 7 — **Delivery (branch → commit → push → PR)**
>
> **Base skill** (profile-neutral). Load together with the profile overlay:
>
> - Frontend: `pr-handoff.fe.md`
> - Backend:  `pr-handoff.be.md`

## Purpose

Turn a converged implementation into a **PR-ready handoff**: exact branch
name, commit sequence, push command, PR title, PR description, and the AC
coverage checklist. **The Developer runs every command.**

## When to use

After Phase 6 (Manual Dev Review & Testing) approves the implementation. If
the Dev requested changes, go back to Phase 5 instead.

## Inputs

- The Implementation Handoff Report (Phase 5).
- The approved ticket (title, ID, four sections).
- The Action Plan's dependency graph (to link related PRs if any).
- Repo branching / commit convention from `AGENTS.md`:
  - Base branch (typical: `development`, `main`, `develop`).
  - Branch prefixes (typical: `feature/`, `fix/`, `chore/`).
  - Commit message format (typical: Conventional Commits with ticket ID).
  - CI hook that enforces the format (if any).

## Steps (base)

1. **Pick the branch prefix.**
   - `feature/` for new capability.
   - `fix/` for bug fixes.
   - `chore/` for maintenance (deps, tests, docs, tooling).
2. **Compose the branch name:** `<prefix>/<TICKET-ID>-<kebab-slug>` where the
   slug is the short imperative title from the ticket, lower-cased and
   kebab-cased. Keep it under ~50 chars.
3. **Compose the commit message(s).** Prefer one commit per PR when the diff
   is small; otherwise a short sequence of Conventional Commits, all
   referencing the ticket ID per the format declared in `AGENTS.md`:
   - `feat(<scope>): <imperative summary> #<TICKET-ID>`
   - `fix(<scope>): <imperative summary> #<TICKET-ID>`
   - `chore(<scope>): <imperative summary> #<TICKET-ID>`
   - `<scope>` matches the primary module / controller / feature affected by
     the change.
4. **Compose the PR title:** exactly the imperative title of the ticket, with
   the ticket ID suffix per `AGENTS.md`:
   `<Ticket title> (#<TICKET-ID>)`.
5. **Compose the PR description** (base template below):
   - Link the Work Item / Issue.
   - AC coverage checklist mirrored from the ticket's `AC` section.
   - Test evidence (which tests cover which AC line).
   - Notes/risks/out-of-scope mirrored from the ticket's `Notes & Obvs`
     section.
   - The exact **manual command list** the reviewer / CI can run.
   - The profile overlay adds a "Contract delta" (BE) or "Design & UX delta"
     (FE) section.
6. **Enforce delivery rules.**
   - Base branch per `AGENTS.md`.
   - PR is focused: **one ticket per PR**. If the diff pulled in unrelated
     changes, split before opening.
   - **No `--force` on shared branches.** **No `--no-verify`.** No editing of
     protected branches.
   - PR must trigger the repo's PR checks (build, test, lint).
7. **Emit the Delivery Bundle** (base template below) and hand it to the
   Developer. **The AI does not run `git` or open the PR.**

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                        | Pass criterion                                                                                              |
|---|------------------------------|-------------------------------------------------------------------------------------------------------------|
| 1 | Branch name valid            | Correct prefix; contains `<TICKET-ID>`; kebab-slug matches the ticket title; sane length.                   |
| 2 | Commits valid                | Conventional Commit format per `AGENTS.md`; every commit references `<TICKET-ID>`; no fixup/wip commits.    |
| 3 | Base branch                  | Base is the branch declared in `AGENTS.md`. Not `main`/`master` unless declared. Not a protected branch.    |
| 4 | Single-ticket PR             | Diff scope matches the ticket; no unrelated changes bundled in.                                             |
| 5 | AC coverage checklist        | Every AC line from the ticket appears in the PR description, each linked to the test(s) that assert it.     |
| 6 | No forbidden flags           | No `--force`, no `--no-verify`, no protected-branch edits, no direct push to the base branch.               |
| 7 | Dev-runnable handoff         | Command list is exact, copy-pasteable, and the AI has not executed any of them.                             |

## DoD / Exit

- Delivery Bundle handed to the Developer. The Dev runs the commands, opens
  the PR against the base branch, links the Work Item / Issue, and requests
  review.
- The PR passes the repo's PR checks.
- Merge is a human action; the AI does not merge.

### Delivery Bundle template (base)

````md
# Delivery Bundle — Ticket #<ID>

- **Profile:** fe | be

## Branch

- **Base:** `<base-branch>`
- **Name:** `<prefix>/<TICKET-ID>-<kebab-slug>`

## Commands (Developer runs — AI must not)

```bash
git fetch origin
git checkout -b <prefix>/<TICKET-ID>-<kebab-slug> origin/<base-branch>

# after applying the diff produced in Phase 5:
git add -A
git commit -m "<type>(<scope>): <imperative summary> #<TICKET-ID>"

# optional local verification before push:
<install>
<build>
<test>

git push -u origin <prefix>/<TICKET-ID>-<kebab-slug>
# then open the PR against `<base-branch>` and link Work Item / Issue #<TICKET-ID>.
```

## PR title

`<Ticket title> (#<TICKET-ID>)`

## PR description

**Work Item / Issue:** #<TICKET-ID>

**Summary**
<one paragraph — what changed and why, mirroring the ticket ToDo>

**AC coverage**
- [x] Given … When … Then … — covered by `<path/to/test>::<method>`.
- [x] Given … When … Then … — covered by `<path/to/test>::<method>`.
- [x] Edge cases / states covered: <list> — see tests.

**Notes / risks / out of scope**
- <mirrored from ticket's Notes & Obvs>

**Pipelines**
- Will run <PR checks name / path> on push.
````

The profile overlay adds a **Contract delta** (BE) or **Design & UX delta**
(FE) section to the PR description template.

## Tools / MCP hooks

- Read the converged diff produced by Phase 5.
- Ticketing MCP (roadmap) — fetch/update the Work Item / Issue; **PR
  creation and git operations remain with the Developer**.
