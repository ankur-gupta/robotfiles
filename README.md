# robotfiles

Global reference files for LLM coding agents.

This repo contains concise, citable tenets for agent behavior in areas where project-local guidance may be absent:

- `tenets/git/COMMITS.md`: commit-message rules for LLM-authored changes.
- `tenets/languages/PYTHON.md`: Python coding tenets, with the Google Python Style Guide as the default external baseline.
- `tenets/languages/references/`: local fallback copies and licenses for external references used by the tenets.

Use these files as global defaults. Project-specific instructions take precedence; when no project-specific guide exists, apply the relevant tenet file before falling back to general conventions or external style guides.

## Start Here

Clone this repository to a stable location:

```sh
git clone git@github.com:ankur-gupta/robotfiles.git "$HOME/robotfiles"
```

Then, choose how much inherited guidance to keep:

1. **Use as-is**. No reset prompt needed.
2. **Keep useful defaults** [Recommended]. For a fresh start that still preserves repository maintenance conventions. Point your coding agent to the location (e.g., `"$HOME/robotfiles"`) and then prompt it with this:

   ```text
   Reset this repository for my own use. Preserve repository meta instructions,
   repo structure, licenses, and agent steering references from files like
   AGENTS.md and CLAUDE.md to the repo's guidance files. Remove personal
   preference tenets, but keep generally useful maintenance guidance such as the
   LLM-authored Git commit-message structure tenet. Do not assume guidance files
   will always live in a directory named "tenets"; preserve references by
   purpose if paths or names change.
   ```

3. **Fresh slate**. Use this when you want only the reusable container and steering hooks. Point your coding agent to the location (e.g., `"$HOME/robotfiles"`) and then prompt it with this:

   ```text
   Reset this repository to a fresh slate for my own use. Preserve repository
   meta instructions, repo structure, the repository license, and agent steering
   references from files like AGENTS.md and CLAUDE.md to the repo's guidance
   files. Remove user preference tenets, plus external reference copies and
   their paired license files when they only support those removed preferences.
   Leave empty or minimal placeholder guidance files where needed so the
   preserved steering references still point somewhere useful. Do not assume
   guidance files will always live in a directory named "tenets"; preserve
   references by purpose if paths or names change.
   ```

## Examples

### Citing a tenet in your codebase

Cite tenets directly in code comments when the reference clarifies a boundary:

```python
# Keep parsing separate from filesystem access; see Tenet 4 [PY-IO-BOUNDARY].
def parse_invoice(raw_invoice: str) -> Invoice:
    ...
```

### Agent Steering

Clone the repository to a known location and point each agent steering file at these inherited defaults:

```markdown
# AGENTS.md
...
Follow `$HOME/robotfiles/tenets/languages/PYTHON.md` for Python guidance unless this project gives a more specific rule.
...
```

```markdown
# CLAUDE.md
...
For LLM-authored commits, follow `$HOME/robotfiles/tenets/git/COMMITS.md` unless this repo has a more specific commit guide.
...
```
