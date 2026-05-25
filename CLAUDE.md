# CLAUDE.md

This repository contains global reference files for LLM coding agents. Keep
changes concise, citable, and easy to reuse from other projects.

## Repository Purpose

- `tenets/git/COMMITS.md` defines global Git commit-message tenets for
  LLM-authored commits.
- `tenets/languages/PYTHON.md` defines global Python coding tenets.
- `tenets/languages/references/` contains local fallback copies and licenses
  for external references used by the tenets.

Project-specific instructions in downstream repositories take precedence over
these global defaults.

## Working Guidelines

- Prefer small, focused changes that preserve the existing tenet structure.
- Before adding a tenet, check whether an existing tenet can be updated instead.
- Keep tenets concrete, stable, and easy to cite.
- Give referable sections stable IDs in square brackets, following the patterns
  already used in the tenet files.
- Cite tenets only when the citation clarifies the reasoning.
- Do not add broad policy or style guidance without a specific reason.

## Git Commits

For LLM-authored Git commits in this repository, follow
`tenets/git/COMMITS.md`.

In particular:

- Use an `llm/<agent>:` subject prefix.
- Describe the meaningful repo-visible change.
- Include machine-parseable trailers such as `Agent:`, `Agent-Authored:`, and
  `Change-Origin:`.
- Include a concise `Scope:` trailer when it helps identify the touched area.
- Do not include user-authored or unrelated changes in an LLM-authored commit.

## Python Guidance

For Python guidance, follow `tenets/languages/PYTHON.md` unless a more specific
project-local Python guide applies.

