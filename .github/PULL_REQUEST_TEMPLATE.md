<!--
Thanks for contributing to ASDD / ai-workflow.

Read `CONTRIBUTING.md` before opening a PR if you have not already. In
particular: SSOT, base + overlays in sync, immutable canonical structure,
and root-cause discipline.

Fill in every section below. Delete none of the headings.
-->

## Summary

<!-- One or two sentences: what does this PR change, and why. -->

## Type of change

<!-- Check all that apply. Delete lines that do not. -->

- [ ] `feat` — new capability (new skill, new profile, new rule)
- [ ] `fix` — bug fix (typo, broken link, wrong step in a skill, template does not parse)
- [ ] `docs` — README / CONTRIBUTING / CHANGELOG / SECURITY only
- [ ] `refactor` — restructuring without behavior change
- [ ] `chore` — repo governance (CI, templates, .gitignore, etc.)
- [ ] `breaking` — changes public interface (skill name, ticket heading, frontmatter shape). **Requires a major version bump.**

## Affected areas

<!-- Check all that apply. -->

- [ ] `ASDD.md` (workflow / phases / gates / principles)
- [ ] `AGENTS.md` (canonical spec / shared rules / frontmatter)
- [ ] `skills/` — which: `<name>`
- [ ] `templates/` — which: `<name>`
- [ ] `prompts/` — which: `<name>`
- [ ] `README.md` / consumer-facing docs
- [ ] Repo governance / CI / templates

## Related issue

<!-- Link the issue this PR addresses. If none exists and this is a
non-trivial change, open one first (see CONTRIBUTING.md). -->

Closes #

## Self-check

- [ ] **SSOT respected.** No rule duplicated across files; I linked instead.
- [ ] **Base + overlays in sync.** If I changed a shared rule, I mirrored
      it in every affected overlay and every affected template.
- [ ] **The 9-section AGENTS.md structure is intact** in every template
      I touched.
- [ ] **The 4-heading ticket format is intact** in every ticket template
      I touched.
- [ ] **All internal links resolve** (relative paths, anchors).
- [ ] **YAML frontmatter parses** in every template with frontmatter I
      touched.
- [ ] **Minimal comments.** No obvious comments, no commented-out prose,
      no TODO / FIXME markers.
- [ ] **Token efficiency.** No preambles, no restated context, no closing
      summaries in the content I added.
- [ ] **No hardcoded org / user names** except in explicit placeholders.
- [ ] **CHANGELOG.md updated** under `[Unreleased]` with a one-line entry.

## Reviewer notes

<!-- Anything specific you want the reviewer to focus on: a subtle
consistency risk, a decision that could go two ways, an edge case you
were unsure about. Delete this section if empty. -->

## Impact on consumers

<!-- If this change affects how consumer projects use ASDD, describe it
here. If the change is transparent (docs, internal restructuring), write
`No consumer-facing impact`. -->
