# Skill: repo-viability-scan

> Phase 1 — **Discovery & Viability**
>
> **Base skill** (profile-neutral). Always load together with the profile
> overlay for the active repo:
>
> - Frontend: `repo-viability-scan.fe.md`
> - Backend:  `repo-viability-scan.be.md`

## Purpose

Confront the Readiness Report from Phase 0 with the **real, current** state
of the consumer repository. Produce a Viability Report with a `GREEN` /
`AMBER` / `RED` rating and a concrete gap list.

## When to use

Immediately after `spec-intake` returns `READY` (either mode). Never skip it
because "the ticket looks simple" — landmines like missing policies,
non-existent routes, or unmapped DTOs are only visible after the scan.

## Inputs

- The Readiness Report emitted by `spec-intake`.
- Read-only access to the whole repository.
- The consumer repo's `AGENTS.md` (canonical code conventions).
- The active ASDD profile (from `AGENTS.md` frontmatter, `asdd/config.yml`,
  or autodetected here).

## Steps (base)

1. **Resolve the ASDD profile.** In order of precedence:
   1. Frontmatter `asdd_profile:` in `AGENTS.md`.
   2. `profile:` field in `asdd/config.yml`.
   3. **Autodetect** using the following heuristics:
      - `package.json` + `vue|react|angular|svelte|@vue/*|next|nuxt|remix|@angular/*|astro` → `fe`.
      - `.csproj`, `.sln`, `pom.xml`, `build.gradle`, `go.mod`, `Cargo.toml`,
        `pyproject.toml`, `Gemfile`, `composer.json` → `be`.
      - **Both FE and BE indicators present in the same repo** → recommend
        `mixed` in the report and ask the human to confirm before continuing
        (do not silently pick one side).
      - Neither, or the human rejects the recommendation → **halt** and
        ask the human to declare the profile explicitly.

   **If the resolved value is `mixed`**, this repo hosts both FE and BE
   code. `mixed` is not a profile of its own — resolve the **effective
   profile for the current ticket** in this order (per
   [`../ASDD.md` → Profiles](../ASDD.md#profiles)):
   1. Per-package `AGENTS.md` (closest-wins). If every touchpoint of the
      ticket falls under one folder that carries its own `AGENTS.md`
      with `asdd_profile: fe` or `asdd_profile: be`, use that profile.
   2. `asdd/config.yml` → `packages:` list — same idea without a physical
      per-package `AGENTS.md`.
   3. Ticket frontmatter `profile: fe` (or `be`) declared at the top of
      the ticket's Markdown.

   If none of the three resolves to a single profile (e.g. the ticket
   spans FE + BE touchpoints and declares no frontmatter), **halt** and
   ask the human to pick one. Never run the overlay for a `mixed` repo
   without a resolved effective profile.
2. **Load the profile overlay** and continue with its Steps.
3. **Detect existing agent tooling.** Before mapping touchpoints, scan the
   consumer repo for pre-existing agent tooling that could conflict with,
   or complement, ASDD. Look for the presence of any of:
   - `AGENTS.md` at the project root (canonical anchor).
   - `CLAUDE.md` at the project root.
   - `.github/copilot-instructions.md` (or `.instructions.md`,
     `.prompt.md` files).
   - `.cursorrules` or `.cursor/rules/*.md` (or `.mdc`).
   - `.agents/skills/*/SKILL.md` (Claude Code SKILL wrappers) or
     `.claude/` folders.
   - MCP configuration: `mcp.json`, `.vscode/mcp.json`, or MCP entries in
     `.mcp/`.

   For each match, record it in the Viability Report's **"Existing
   tooling"** section (see template below). Rating impact:
   - No matches → no impact.
   - Matches that already point at or extend `asdd/ai-workflow/` → no
     impact.
   - Matches that carry rules potentially conflicting with ASDD (own
     ticket format, own state boundaries, own auth policies) → **AMBER**
     with a note pointing the human to the README's
     [Coexisting with existing setups](../README.md#coexisting-with-existing-setups)
     section.
   - Duplicate-name Claude SKILL wrappers for any of the seven ASDD skills
     → **AMBER** with an explicit "pick one, delete/rename the other"
     recommendation.
4. **Map touchpoints** — the overlay lists the exact paths to search per
   profile (controllers/services/repos for BE; components/stores/composables
   for FE).
5. **Diff desired vs current.** For each touchpoint output one of:
   - `EXISTS_OK` — matches spec, no change needed.
   - `EXISTS_NEEDS_CHANGE` — describe the delta.
   - `MISSING` — describe what needs to be created and in which layer/module.
6. **Flag landmines.** Explicitly check each of the repo-wide rules declared
   in `AGENTS.md` against the proposed change. The overlay lists the
   profile-typical landmines.
7. **Rate viability:**
   - `GREEN` — every touchpoint is either `EXISTS_OK` or a well-scoped
     `MISSING` / `EXISTS_NEEDS_CHANGE` inside the current conventions. No open
     questions.
   - `AMBER` — one or more open questions that do not block starting Phase 2,
     but must be answered before Phase 5 begins.
   - `RED` — a blocker: undecided schema-change flow, missing auth policy,
     breaking change, or a decision that only a human owner can make.
     **Escalate immediately with the exact question.**
8. **Emit the Viability Report** (base template below; the overlay may add
   sections) and hand off to Phase 2 (`action-plan-builder`) unless `RED`.

## Loop

Apply [`../prompts/loop-self-verify.md`](../prompts/loop-self-verify.md) with
the phase override below and the profile overlay's rubric additions.

**Phase override (base):**

| # | Check                       | Pass criterion                                                                                        |
|---|-----------------------------|-------------------------------------------------------------------------------------------------------|
| 1 | Profile resolved            | The active profile is declared explicitly (frontmatter, config, or human choice); autodetection notes recorded. |
| 2 | Existing tooling detected   | Pre-existing agent tooling (AGENTS.md, CLAUDE.md, Copilot / Cursor / Claude configs, MCP) is enumerated in the report; conflicts flagged as AMBER with a pointer to the README's coexistence section. |
| 3 | Touchpoint coverage         | Every AC line has at least one mapped touchpoint.                                                     |
| 4 | Reality check               | Every `EXISTS_*` claim references a real file path in the repo (not invented).                        |
| 5 | Landmine sweep              | Every AGENTS.md-declared repo-wide rule is reviewed against the change.                               |
| 6 | Open questions explicit     | Every `AMBER`/`RED` item names the exact decision required and the human owner.                       |
| 7 | Handoff to Phase 2          | The Action Plan builder can slice from this report without re-scanning the repo.                      |

## DoD / Exit

- Viability Report emitted, rating one of `GREEN` / `AMBER` / `RED`.
- `RED` → **do not** advance. Escalate to the human with the decision required.
- `GREEN` / `AMBER` → hand off to `action-plan-builder`.

### Viability Report template (base)

```md
# Viability Report — <feature name>

- **Profile:** fe | be  (source: frontmatter | config | autodetect | human)
- **Rating:** GREEN | AMBER | RED
- **Source Readiness Report:** <link/section>

## Existing agent tooling

| Kind | Path | Impact | Notes |
|------|------|--------|-------|
| AGENTS.md (project root) | `./AGENTS.md` | none / AMBER | conflict / merge notes |
| CLAUDE.md | `./CLAUDE.md` | none / AMBER | ... |
| Copilot instructions | `.github/copilot-instructions.md` | none / AMBER | ... |
| Cursor rules | `.cursor/rules/*` or `.cursorrules` | none / AMBER | ... |
| Claude SKILL wrappers | `.agents/skills/<name>/SKILL.md` | none / AMBER | duplicate-name warnings |
| MCP config | `mcp.json` / `.vscode/mcp.json` | none | list available MCPs |

If any row is AMBER, point the human to
`README.md` → *Coexisting with existing setups* before Phase 2 starts.

## Touchpoints

| Layer / concern | Path | Status | Notes |
|-----------------|------|--------|-------|
| <layer 1> | `<path>` | EXISTS_OK / EXISTS_NEEDS_CHANGE / MISSING | … |
| <layer 2> | `<path>` | … | … |

## Landmines / repo rules

- <AGENTS.md rule 1>: …
- <AGENTS.md rule 2>: …

## Open questions

- [AMBER] <question> — owner: <who> — required before Phase 5.
- [RED]   <question> — owner: <who> — blocker for Phase 2.

## Gap list

- Create/modify `<file>` to <effect>.
- Add/rename `<method / DTO / component / store>`.
```

## Tools / MCP hooks

- Repository search / read (glob + text search over the paths listed in Inputs).
- Language servers (references, definitions) when available.
- Profile overlay may add MCP hooks (Swagger endpoint for BE, Figma MCP for FE).
