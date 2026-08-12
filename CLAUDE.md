# CLAUDE.md

**Read [AGENTS.md](./AGENTS.md) before planning, coding, or answering.**
This file contains no rules of its own — it exists only because
**Claude Code** reads `CLAUDE.md` by default.

## Where things live in this repo (`ai-workflow/`)

- [`ASDD.md`](ASDD.md) — the master workflow (Spec → Scan → Plan → Ticket →
  Build → PR), phases, gates, principles.
- [`AGENTS.md`](AGENTS.md) — the canonical spec of what an ASDD-compatible
  `AGENTS.md` must contain, plus shared workflow rules.
- [`prompts/loop-self-verify.md`](prompts/loop-self-verify.md) — the
  self-verification engine every artifact runs through.
- [`skills/`](skills/) — the seven phase skills (neutral base
  `<name>.md` + profile overlays `<name>.fe.md` / `<name>.be.md`).
- [`templates/`](templates/) — ready-to-copy templates:
  - `AGENTS.md.fe.example` / `AGENTS.md.be.example` — profile-specific
    starting points for the consumer's `AGENTS.md`.
  - `ticket.fe.md` / `ticket.be.md` — the fixed 4-heading ticket format.
  - `CLAUDE.md.example` / `copilot-instructions.md.example` /
    `cursor-rules.md.example` — shims that consumers copy into their repo.

## Note on scope

`ai-workflow/` is a **methodology repo**, not a product repo. This
`CLAUDE.md` and the sibling `AGENTS.md` describe **how to work with the
ASDD methodology itself** (authoring skills, editing templates,
maintaining SSOT). They are **not** what a consumer repo copies — the
consumer copies the templates from `templates/` (see
[`README.md`](README.md) for the install steps).
