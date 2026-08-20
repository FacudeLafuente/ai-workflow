---
name: Bug report
about: A skill, template, or prompt produces wrong output, or a documented step does not work.
title: "bug: <one-line summary>"
labels: ["bug", "triage"]
---

## What is broken

<!-- One or two sentences: which file / skill / template is at fault, and
what wrong behavior you observed. -->

## Where

<!-- Path(s) in this repo. Include line numbers if you have them. -->

- File(s):
- Line(s):
- ASDD phase (if applicable):

## Steps to reproduce

<!-- Enough for a maintainer to reproduce without asking. If the bug is
in an AI-agent-produced artifact, include the trigger phrase, the
consumer repo shape, and the profile (`fe` / `be` / `mixed`). -->

1.
2.
3.

## Expected behavior

<!-- What the file / skill / template should have produced. Cite the
canonical source of truth for that behavior (e.g. `ASDD.md` line X,
`AGENTS.md` line Y, `loop-self-verify.md` rubric row Z). -->

## Actual behavior

<!-- What you got instead. Paste the failing output if it fits. -->

## Environment

- **Agent tool:** GitHub Copilot / Claude Code / Cursor / Codex / other
- **Profile:** fe / be / mixed
- **ASDD version / commit:** `<tag or SHA of asdd/ai-workflow/>`
- **Consumer project stack (if relevant):** `<one line>`

## Screenshots / logs (optional)

<!-- Attach relevant screenshots or paste error output inside a fenced
code block. Redact anything client-confidential. -->

## Additional context

<!-- Workarounds you tried, adjacent issues, guesses about root cause. -->
