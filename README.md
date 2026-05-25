# robotfiles

Global reference files for LLM coding agents.

This repo contains concise, citable tenets for agent behavior in areas where project-local guidance may be absent:

- `tenets/git/COMMITS.md`: commit-message rules for LLM-authored changes.
- `tenets/languages/PYTHON.md`: Python coding tenets, with the Google Python Style Guide as the default external baseline.
- `tenets/languages/references/`: local fallback copies and licenses for external references used by the tenets.

Use these files as global defaults. Project-specific instructions take precedence; when no project-specific guide exists, apply the relevant tenet file before falling back to general conventions or external style guides.

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
