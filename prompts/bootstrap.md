# Prompt: ASDD Bootstrap

> Reusable prompt for **kicking off ASDD in a consumer repo**. Any
> ASDD-aware agent should follow this file when the user issues a
> bootstrap trigger phrase (see below).

---

## Trigger phrases

Any of these, exact or case-insensitive, activate this prompt:

- `Start ASDD`
- `Iniciar ASDD`
- `Run ASDD`
- `asdd init`

## Purpose

Detect the **connection layer** the consumer needs at the project root,
report what is present / missing / needs merge, and **list** the exact
safe-copy commands for the user to run. The agent does **not** execute
those commands — see the guardrail below.

## When to use

- Right after the user cloned `ai-workflow` into `asdd/ai-workflow/`
  (see [`../README.md` → Getting started](../README.md#getting-started)),
  as the first interaction with an ASDD-aware agent.
- Any time the user wants to verify the setup is complete.

## Guardrail (non-negotiable)

- The AI **never** runs `cp`, `mkdir`, `git`, `chmod`, or any other
  console command during bootstrap. This is ASDD Principle #8 (see
  [`../ASDD.md` → Principles](../ASDD.md#principles)).
- The AI **lists** the exact commands in the report. The Developer runs
  them.
- Detection is **read-only**: file existence checks, frontmatter reads,
  optional grep for entries in `.gitignore`.

## Detection steps

Execute each read-only check and record the result for the report.

### 1. Locate context

- Confirm the current working directory ends with
  `.../asdd/ai-workflow`. If not, ask the user to clarify where the
  clone lives; do not guess.
- Compute the **project root** as two levels up from the clone:
  `../../` relative to `asdd/ai-workflow/`.

### 2. Required — project state

- **`<project-root>/.git/`** — exists? Report `git repo` / `NOT a git
  repo` (a project that is not a git repo can still use ASDD, but the
  Delivery phase will require `git init` first).
- **`<project-root>/AGENTS.md`** — exists? If yes:
  - Read its frontmatter. Record `asdd_profile:` value if present.
  - If frontmatter is missing or `asdd_profile:` is not declared, mark
    as **AMBER — needs merge with ASDD frontmatter**.
- **`<project-root>/.gitignore`** — contains a line matching `asdd/`?
  If missing or absent, mark as **missing**.

### 3. Required — profile resolution

Resolve the active profile in this order of precedence (matches
[`../ASDD.md` → Profiles](../ASDD.md#profiles)):

1. **`asdd_profile:`** in `<project-root>/AGENTS.md` frontmatter.
2. **`profile:`** in `<project-root>/asdd/config.yml`.
3. **Autodetection heuristics** — reuse the same heuristics documented in
   the `repo-viability-scan` skill:
   - FE: `package.json` + `vue|react|angular|svelte|@vue/*|next|nuxt|remix|@angular/*|astro`, or React Native / Expo / Ionic.
   - BE: `.csproj`, `.sln`, `pom.xml`, `build.gradle`, `go.mod`,
     `Cargo.toml`, `pyproject.toml`, `Gemfile`, `composer.json`.
   - Ambiguous → mark **AMBER — ask the human**.

### 4. Optional — connection-layer shims

Report each as `present` / `not present` / `AMBER — needs merge` (per the
README's [Coexisting with existing setups](../README.md#coexisting-with-existing-setups)):

- **`<project-root>/CLAUDE.md`** (relevant if the team uses Claude Code).
- **`<project-root>/asdd/config.yml`** (relevant for monorepos with
  per-package overrides).

### 5. Optional — existing agent tooling

Detect and report any of these; conflicts are AMBER, pointer to the
README's coexistence section:

- `<project-root>/AGENTS.md` without ASDD frontmatter or missing
  canonical sections.
- `<project-root>/.cursorrules` or `.cursor/rules/*.md`.
- `<project-root>/.github/copilot-instructions.md` without ASDD block.
- `<project-root>/.agents/skills/*/SKILL.md` (Claude wrappers), especially
  ones colliding with the seven ASDD skill names.
- `<project-root>/mcp.json` or `<project-root>/.vscode/mcp.json`.

This detection duplicates the intent of Step 3 in
[`../skills/repo-viability-scan.md`](../skills/repo-viability-scan.md).
It runs here as a first-touch guard; the full scan runs later at Phase 1.

## Report template

Emit the report **verbatim** in this shape (use the actual detected
values):

```md
# ASDD Bootstrap Check

- **Project root:** `<absolute path>`
- **Clone location:** `<absolute path to asdd/ai-workflow>`
- **Detected profile:** fe | be | ambiguous
- **Detected source:** frontmatter | config.yml | autodetect | none

## Required

| Item                                | Status                          |
|-------------------------------------|---------------------------------|
| `.git/` at project root             | ✅ present / ❌ missing         |
| `AGENTS.md` at project root         | ✅ present / ❌ missing         |
| `AGENTS.md` frontmatter with profile| ✅ ok / ⚠️ needs merge / ❌ missing |
| `.gitignore` contains `asdd/`       | ✅ present / ❌ missing         |
| Profile resolvable                  | ✅ (fe/be) / ⚠️ ambiguous       |

## Optional

| Item                                | Status                          |
|-------------------------------------|---------------------------------|
| `CLAUDE.md`                          | present / not present           |
| `asdd/config.yml`                    | present / not present (only needed for monorepos) |

## Existing tooling detected

| Kind                                 | Path                            | Impact |
|--------------------------------------|---------------------------------|--------|
| (list only rows that exist)          |                                 |        |

If any row above is present without ASDD frontmatter / pointer, mark
**AMBER** and link to `../README.md#coexisting-with-existing-setups`.

## Recommended actions

**The Developer runs the commands below. The AI does not.**

<Emit only the commands needed to fix the missing / AMBER items,
using the safe-copy pattern from the README. Skip any command whose
target is already correct.>

```bash
# Example — only include if needed:
echo "asdd/" >> .gitignore
git add .gitignore
git commit -m "chore: ignore local ASDD workspace"

# Example — only include if AGENTS.md is missing:
[ -e ./AGENTS.md ] \
  && cp asdd/ai-workflow/templates/AGENTS.md.<profile>.example ./AGENTS.md.new \
  || cp asdd/ai-workflow/templates/AGENTS.md.<profile>.example ./AGENTS.md
```

## Next step

After running the commands, run `Start ASDD` again to verify. When every
Required row is `✅`, the workflow is ready for Phase 0 (`spec-intake`).
See [`../ASDD.md` → Flow overview](../ASDD.md#flow-overview).
```

## Exit conditions

- **READY** — every Required row is `✅`, profile is resolvable, no AMBER
  in the Existing tooling table → the agent replies with the report and
  tells the user "ready to start Phase 0; give me the feature or
  `Mode B` ticket ID".
- **NEEDS ACTION** — one or more Required rows are `❌` or `⚠️`, or the
  profile is ambiguous → the agent replies with the report + the exact
  commands for the human to run, and stops. Do not attempt to run them.

## Anti-patterns (auto-fail the bootstrap)

- Executing any command. Detection is read-only.
- Guessing the profile when it is ambiguous. Ask the user.
- Overwriting existing files. Always use the safe-copy `[ -e … ] && …
  .new || …` pattern.
- Duplicating rule content in the report. The report links to
  [`../README.md`](../README.md) and [`../AGENTS.md`](../AGENTS.md) for
  the actual rules (SSOT).
