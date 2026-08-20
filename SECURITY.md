# Security Policy

## Scope

This repository is a **methodology repo** — Markdown documents, skill
prompts, and templates. It ships no runtime code, no dependencies, no
network endpoints. As a result, the classic attack surface (RCE, XSS,
SQLi, dependency CVEs) does not apply directly.

The security concerns that **do** apply are:

- **Prompt-injection payloads** hidden inside skills, templates, or
  prompts that could hijack an AI agent operating in a consumer repo.
- **Rule / guardrail bypasses** introduced through subtle wording changes
  (e.g. a skill that silently permits `--no-verify`, `git push --force`,
  or executing shell commands the AI is not supposed to run).
- **Leakage of internal, proprietary, or client-confidential content**
  inside examples, sample tickets, or prose.
- **Malicious or misleading external links** in documentation.

## Reporting a vulnerability

**Do not open a public GitHub issue for a security concern.**

Instead, contact the maintainers privately:

- **Primary contact:** `@FacudeLafuente` (via internal Slack DM,
  channel `#asdd`, or corporate email).
- **Escalation:** if you cannot reach the primary contact within 48
  hours, escalate through the AI-Skills maintainers listed at
  [`Alix-Platforms/AI-Skills` → CODEOWNERS](https://github.com/Alix-Platforms/AI-Skills/blob/main/.github/CODEOWNERS).

Please include:

- The file(s) and lines involved.
- A description of the concern (prompt injection vector, guardrail
  bypass, leaked content, etc.).
- If applicable, a proof-of-concept payload or the observed agent
  behavior.

## Response expectations

- Acknowledgement within **2 business days**.
- Triage and mitigation plan within **5 business days**.
- Public disclosure only after a fix has landed on `main` and the
  consumers have been notified through the CHANGELOG.

## Non-goals

- This policy does not cover vulnerabilities in the **consumer repos**
  that adopt ASDD — those are the responsibility of each consumer team.
- This policy does not cover vulnerabilities in third-party AI agents
  (GitHub Copilot, Claude Code, Cursor, Codex) that read ASDD skills —
  report those to the respective vendor.
