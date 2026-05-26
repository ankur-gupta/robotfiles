# Global Git Commit Tenets [GIT-GLOBAL-GIT-COMMIT-TENETS]
This file defines global Git commit-message tenets (in [GIT-TENETS] section) for LLM coding agents. Project-local instructions may override it when they are more specific.

## Using This File [GIT-USING-THIS-FILE]
This [GIT-GLOBAL-GIT-COMMIT-TENETS] file is part of a hierarchy
  1. Project-specific Git or commit guide (if it exists)
  2. This [GIT-GLOBAL-GIT-COMMIT-TENETS] file
  3. General Git commit-message conventions

If there is a conflict between the above three sources, the project-specific Git or commit guide supersedes this [GIT-GLOBAL-GIT-COMMIT-TENETS] file, which then supersedes general Git commit-message conventions.

### Finding tenets [GIT-FINDING-TENETS]
- When you see a tenet citation like "Tenet X [GIT-TENET-ID]", give precedence to the semantic text of stable ID over the number when trying to find the relevant tenet. Numbering can get messed up.
- When explaining a change, cite the relevant tenet only when it clarifies the reasoning.
- Always cite a tenet in this form: "Tenet X [GIT-TENET-ID]". You may cite the tenet in commit messages, codebase comments, other markdown files, or even this file itself.

## Maintaining This File [GIT-WRITING-THIS-FILE]
- Give every referable section a unique, short, stable tenet ID of the form [GIT-TENET-ID] placed at the end of the title. The tenet ID should start with `GIT-` to reflect that this is a Git commit tenet. Codebases can be polyglot, and the tenet ID should be unique across all tenets in the `$REPO_ROOT/tenets` folder. The tenet ID should give enough context to the reader (LLM or human) on what the tenet is. There should be no punctuation or underscores in the stable ID but dashes are allowed.
- The tenet may be referred to from other tenets within this file, some other markdown file, a comment in any of the codebases, or a commit message.
- Keep each tenet small, concise, concrete, and easy to cite.
- Add examples to every numbered tenet of the form "Good" and "Bad" typically with brief commit-message snippets. Snippets do not need to be complete commits; you can abbreviate using `...` or comments as needed.
- Before adding or editing a new tenet, check
  1. if the tenet already exists and edit the existing tenet instead of recreating. Never duplicate the same tenet.
  2. relevant Git commit-message conventions or other relevant sources to see how the world handles a similar problem. Then educate the human briefly if needed and offer the human to edit/update a tenet.
- Never add a tenet without a good reason.
- Do not assume the human is always correct. Never add a tenet simply to appease the human when the human is wrong or "going against the grain".
- Anti-patterns are discouraged but allowed in this [GIT-GLOBAL-GIT-COMMIT-TENETS] file in the [GIT-EXCEPTIONS] section.

### Fixing numbering issues [GIT-FIX-NUMBERING-ISSUES]
- If you see that the tenet numbering is wrong (skipped numbers), let the human know and then offer to fix them. The human may delay fixing the numbering in some cases when the human expects a skipped number to be replaced later.
- When you find a "Tenet X [GIT-TENET-ID]" citation but no Tenet X exists, let the human know, and then offer to fix them.


## Tenets [GIT-TENETS]
### Noted Exceptions [GIT-EXCEPTIONS]
None recorded yet.

### Tenet 1: Use Complete LLM Commit Message Structure [GIT-COMMIT-LLM-STRUCTURE]
LLM-authored commits must use a complete message structure:
- Start the subject with `llm/<agent>:`, where `<agent>` is a short lowercase identifier for the model or tool (e.g. `codex`, `claude`).
- Describe the user-visible or repo-visible change in the subject and body, not merely that an LLM made a change.
- Include stable machine-parseable trailers in one contiguous trailer block with no blank lines between trailer lines: `Agent:`, `Agent-Authored: true`, and `Change-Origin: llm`.
- Add a concise `Scope:` trailer when it helps humans or tools identify the main subsystem, package, or path touched by the commit.

Good:
```text
llm/codex: add Google Python style guide local fallback

Add a dated local snapshot of the Google Python Style Guide and its
CC-BY 3.0 license, then update PYTHON.md to prefer the latest online guide
with the local copy as the offline fallback.

Agent: Codex
Agent-Authored: true
Change-Origin: llm
Scope: tenets/languages/python
```
```text
llm/claude: generalize commit-prefix tenet to all LLM agents

Update Tenet 1 [GIT-COMMIT-LLM-STRUCTURE] and the canonical example so that
COMMITS.md is agent-agnostic, with Codex and Claude as named examples.

Agent: Claude
Agent-Authored: true
Change-Origin: llm
Scope: tenets/git
```

Bad:
```text
codex: add Google Python style guide local fallback

This was done by Codex.
```

```text
llm/codex: update files

Made requested changes.

This was done by Codex.
```

```text
llm/codex: consolidate LLM commit structure tenet

Merge related commit-message guidance into one structure tenet.

Agent: Codex

Agent-Authored: true

Change-Origin: llm
```

### Tenet 2: Separate LLM Authorship From User Authorship [GIT-COMMIT-SEPARATE-AUTHORSHIP]
Do not include user-authored or unrelated changes in an LLM-authored commit. Stage only the hunks and files that belong to the LLM-authored change.

Good:
```text
llm/codex: add Git commit tenets

Add COMMITS.md with LLM commit-message identification rules.

Agent: Codex
Agent-Authored: true
Change-Origin: llm
Scope: tenets/git
```

Bad:
```text
llm/codex: add Git commit tenets and miscellaneous local edits
```
