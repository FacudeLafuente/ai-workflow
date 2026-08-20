# Changelog

All notable changes to ASDD / ai-workflow are documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Consumer projects clone this repo into `asdd/ai-workflow/` and pin a tag
(or track `main` at their own risk). Every **breaking change** requires
a **major** version bump and a migration note under the release.

## [Unreleased]

### Added

- Repo governance layer: `LICENSE` (proprietary), `SECURITY.md`,
  `CONTRIBUTING.md`, `CHANGELOG.md`, `.github/CODEOWNERS`,
  `.github/PULL_REQUEST_TEMPLATE.md`, `.github/ISSUE_TEMPLATE/`
  (bug / feature / question / config), `.editorconfig`, `.gitattributes`,
  and a base `.gitignore`.

### Changed

- `README.md`: replaced `<org>` placeholder with `Alix-Platforms` in the
  clone command.
- `skills/repo-viability-scan.md`: profile-resolution algorithm now
  covers `asdd_profile: mixed`, including the per-ticket effective
  profile resolution (per-package `AGENTS.md` closest-wins →
  `asdd/config.yml` `packages:` → ticket frontmatter) and the halt
  behavior when the effective profile is not resolvable.
- `skills/spec-intake.md`: added `mixed`-repo input requirement (ticket
  effective profile must be provided or the skill halts) and the
  `Mixed-repo effective profile source` field in the Readiness Report.
- `skills/ticket-author.md`: added Step 3 to emit the `profile:` YAML
  frontmatter in tickets whenever the consumer repo is `mixed` and the
  profile is not resolvable from a per-package `AGENTS.md` or
  `config.yml` `packages:` entry. Mode B validation now cross-checks
  this.
- `skills/ticket-quality-gate.md`: added rubric row `#2 Mixed-repo
  profile declared` to reject tickets in `mixed` repos that do not
  resolve to a single effective profile.

## [0.9.0] — 2026-08-20

Initial internal beta. Ready for pilot adoption by 2–3 teams before a
`1.0.0` release cut.

### Added

- **Master workflow** in [`ASDD.md`](ASDD.md): 8 phases, 2 human gates,
  self-verification loop, 9 core principles.
- **Canonical `AGENTS.md` spec** in [`AGENTS.md`](AGENTS.md): 9-section
  structure, required frontmatter (`asdd_profile`, `base_branch`,
  `ticketing`), and 8 shared rules every consumer must enforce
  (no-commands, root-cause, ticket format, SSOT, delivery, minimal
  comments, token efficiency, bootstrap triggers).
- **Seven skills** for every phase, each with neutral base +
  profile overlays (`skills/<name>.md` / `.fe.md` / `.be.md`):
  `spec-intake`, `repo-viability-scan`, `action-plan-builder`,
  `ticket-author`, `ticket-quality-gate`, `implementation-loop`,
  `pr-handoff`.
- **Self-verification prompt** in
  [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md) — the
  rubric every artifact runs through until convergence.
- **Bootstrap prompt** in [`prompts/bootstrap.md`](prompts/bootstrap.md)
  — read-only detection of the connection layer, triggered by
  `Start ASDD` / `Iniciar ASDD` / `Run ASDD` / `asdd init`.
- **Profile templates** in [`templates/`](templates/):
  - `AGENTS.md.fe.example` — Frontend starting point (Vue 3 baseline).
  - `AGENTS.md.be.example` — Backend starting point (.NET 10 baseline).
  - `AGENTS.md.mixed.example` — FE + BE in the same repo (monorepo
    with separate packages, or monolith with both intermingled).
  - `ticket.fe.md` — FE ticket format (`ToDo / AC / Links to Figma /
    Notes & Obvs`).
  - `ticket.be.md` — BE ticket format (`ToDo / AC / Contracts & Specs /
    Notes & Obvs`).
- **Agent shims** in [`templates/`](templates/):
  `CLAUDE.md.example`, `copilot-instructions.md.example`,
  `cursor-rules.md.example`.
- **`mixed` profile support**: repos hosting both FE and BE code declare
  `asdd_profile: mixed`; the effective profile is resolved **per
  ticket** via per-package `AGENTS.md`, `asdd/config.yml` `packages:`,
  or the ticket's own `profile:` frontmatter. See
  [`ASDD.md` → Profiles](ASDD.md#profiles).
- **Repo governance**: `LICENSE` (proprietary), `SECURITY.md`,
  `CONTRIBUTING.md`, `.github/CODEOWNERS`, `.github/ISSUE_TEMPLATE/`,
  `.github/PULL_REQUEST_TEMPLATE.md`, `.editorconfig`, `.gitattributes`.

### Notes

- This is a **pre-1.0 beta**. Public interfaces (skill names, ticket
  headings, frontmatter shape) may still change without a major bump
  until `1.0.0` is cut.
- Consumers who need stability should pin the `v0.9.0` tag.

[Unreleased]: https://github.com/Alix-Platforms/ai-workflow/compare/v0.9.0...HEAD
[0.9.0]: https://github.com/Alix-Platforms/ai-workflow/releases/tag/v0.9.0
