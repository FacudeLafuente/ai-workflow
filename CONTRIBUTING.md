# Contributing to ASDD / ai-workflow

Thanks for taking the time to improve ASDD. This repo is the **portable
core of the methodology** — every change here ripples into every consumer
project that clones it under `asdd/ai-workflow/`. That is a feature, not a
bug, but it raises the bar for what lands on `main`.

Read this file end-to-end before opening your first PR.

## Ground rules

1. **Single Source of Truth (SSOT).** Never duplicate a rule that already
   lives somewhere else in the repo. Link to the canonical location.
   Duplication is a defect the reviewer will flag.
2. **Base + overlays stay in sync.** If you edit a shared rule in
   [`AGENTS.md`](AGENTS.md), mirror it in **every** profile template
   under [`templates/`](templates/) and in every affected skill under
   [`skills/`](skills/). Same for [`ASDD.md`](ASDD.md).
3. **The 9-section canonical structure of `AGENTS.md` is immutable.** So
   is the 4-heading ticket format
   ([`templates/ticket.fe.md`](templates/ticket.fe.md) /
   [`templates/ticket.be.md`](templates/ticket.be.md)). Do not rename,
   reorder, or add extra top-level headings.
4. **Root-cause, not shortcuts.** Do not silence warnings, delete tests,
   narrow scope, or bypass the loop to make a PR mergeable. See
   [`AGENTS.md` → Rule #2](AGENTS.md#2-root-cause-not-shortcuts).
5. **Minimal comments.** Follow
   [`AGENTS.md` → Rule #6](AGENTS.md#6-minimal-comments-in-generated-code)
   when writing prose in skills, templates, and prompts.
6. **Token efficiency.** Follow
   [`AGENTS.md` → Rule #7](AGENTS.md#7-token-efficiency). The consumer
   loads these files into their agent's context on every session — every
   extra line has a cost.

## What kind of change goes where

| Change | Where it belongs |
|---|---|
| Add / change a phase, gate, or principle | [`ASDD.md`](ASDD.md) |
| Add / change the AGENTS.md spec (structure, frontmatter, shared rules) | [`AGENTS.md`](AGENTS.md) + mirror in every `templates/AGENTS.md.*.example` |
| Add / change a skill's behavior | [`skills/<name>.md`](skills/) (base first, then `.fe` / `.be` overlays) |
| Add / change the ticket format | [`templates/ticket.<profile>.md`](templates/) |
| Add / change the self-verification rubric | [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md) |
| Add / change the bootstrap detection | [`prompts/bootstrap.md`](prompts/bootstrap.md) |
| Add a new profile (`data`, `infra`, …) | Discuss in an issue first — this is a spec-level change that touches every skill and every template |
| Fix a typo / broken link / example | Straight PR |

## Workflow

### 1. Open an issue first for anything non-trivial

Bug fixes and typo fixes go straight to a PR. Anything that touches
`ASDD.md`, `AGENTS.md`, a skill's behavior, or the ticket format needs
an issue first so the design can be discussed before implementation.

Templates: see [`.github/ISSUE_TEMPLATE/`](.github/ISSUE_TEMPLATE/).

### 2. Branch and commit

- Branch off `main`: `<prefix>/<short-slug>` where prefix is one of
  `feat`, `fix`, `docs`, `refactor`, `chore`.
- Conventional commits:

  ```text
  <type>(<scope>): <lowercase description>
  ```

  - Allowed types: `feat | fix | docs | style | refactor | test | chore
    | perf | ci | build | revert`.
  - Scopes we use: `asdd`, `agents`, `skill`, `template`, `prompt`,
    `readme`, `ci`.

### 3. Self-check before opening the PR

Run through this checklist yourself before requesting review:

- [ ] **SSOT respected.** No rule duplicated across files.
- [ ] **Base + overlays in sync.** If I changed a shared rule, I checked
      every overlay and every template.
- [ ] **The 9-section AGENTS.md structure is intact** in every template I
      touched.
- [ ] **The 4-heading ticket format is intact** in every ticket template
      I touched.
- [ ] **All internal links resolve** (relative paths, anchors).
- [ ] **YAML frontmatter parses** in every template with frontmatter
      (`AGENTS.md.*.example`, `ticket.*.md` if applicable).
- [ ] **No hardcoded org / user names** except in explicit placeholders.
- [ ] **CHANGELOG.md updated** under `[Unreleased]` with a one-line entry.

### 4. PR

- Use [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md).
- One conceptual change per PR. Split refactors from behavior changes.
- Wait for CODEOWNERS review. Do not merge your own PR unless explicitly
  agreed with a co-maintainer.
- After merge, if the change is user-facing, update the CHANGELOG's
  `[Unreleased]` block into a new versioned release entry.

## Reviewer guidance

- Enforce SSOT ruthlessly. If a rule is stated in two places, ask for one
  to be removed and replaced with a link.
- Check base ↔ overlay ↔ template consistency.
- Verify the change does not regress the token-efficiency rule
  ([`AGENTS.md` → Rule #7](AGENTS.md#7-token-efficiency)).
- If the PR touches `ASDD.md`, `AGENTS.md`, or a skill's rubric, cross-check
  that the self-verification loop
  ([`prompts/loop-self-verify.md`](prompts/loop-self-verify.md)) still
  aligns.

## Security

Do not open a public issue for security concerns. See
[`SECURITY.md`](SECURITY.md).

## Questions

- Ask in an issue with the `question` template.
- For real-time chat, use the internal Slack channel `#asdd` (or ping the
  CODEOWNERS).
