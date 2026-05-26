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
Follow `$HOME/robotfiles/tenets/languages/PYTHON.md` for Python guidance 
unless this project gives a more specific rule.
...
```

```markdown
# CLAUDE.md
...
For LLM-authored commits, follow `$HOME/robotfiles/tenets/git/COMMITS.md` 
unless this repo has a more specific commit guide.
...
```

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

## Semantic Pull (like `git pull`)

Use this when you have your own copy of these tenets and want an LLM to pull in
new upstream guidance from
[ankur-gupta/robotfiles](https://github.com/ankur-gupta/robotfiles) without
overwriting your local preferences. A semantic pull compares intents, not just
lines: it imports generally useful new tenets, preserves project- or
user-specific choices, and asks before changing guidance that appears
intentionally local. The LLM agent should make proposed changes on a separate
branch so you can inspect the diff in your editor before pulling them into your
main working branch.

Open your local copy of this repository in your coding agent, then paste this
prompt:

````text
Perform a semantic pull from the upstream `robotfiles` repository into this
repository, which is my local copy.

First, inspect the local repository status:

```sh
git status --short
```

Do not overwrite, discard, reset, checkout away, stash, or otherwise destroy
uncommitted local changes. Do not run destructive commands such as
`git reset --hard` or `git checkout -- .` as part of this workflow. If
uncommitted changes are present, stop and ask me how to proceed before creating
a branch, downloading upstream, or editing files.

Next, download or update upstream in `/tmp/robotfiles-upstream`. Run only the
command sequence that matches the current filesystem state:

```sh
# If `/tmp/robotfiles-upstream` does not exist, clone it.
git clone --depth 1 --branch main git@github.com:ankur-gupta/robotfiles.git /tmp/robotfiles-upstream
```

```sh
# If `/tmp/robotfiles-upstream` exists, update it.
git -C /tmp/robotfiles-upstream fetch origin main
git -C /tmp/robotfiles-upstream checkout main
git -C /tmp/robotfiles-upstream pull --ff-only origin main
```

Use `/tmp/robotfiles-upstream` as the upstream source. Treat the upstream clone
as a source of reusable default guidance, not as an authority over my local
choices. Compare files by purpose and stable IDs, not only by path or line
diff.

Present pre-change summary: first, analyze the differences and present a summary of changes to the user in these categories:
1. New upstream tenets that are not present in my local copy.
2. Updated, non-conflicting upstream tenets that appear to correspond to existing local tenets.
3. Conflicting tenets where upstream and local guidance disagree or local customization may be intentional.

Make sure to highlight these issues in the summary:
1. Any copyright or licensing issues, including new or modified licenses and
   incompatible licenses.
2. The importance of implementing this semantic pull. Minor, cosmetic upstream
   changes may not need to be pulled.

Present Update Options: second, present the following update options to the user and get explicit approval before proceeding. 
1. Don't apply any changes and stop.
2. [Recommended] Add new tenets only. Do not update any existing tenets.
3. Add new tenets and update non-conflicting tenets. Do not update any
   conflicting tenets.
4. Add new tenets, update non-conflicting tenets, and replace conflicting
   tenets over my local copy.
All update options, even the most drastic change, must make changes in a
separate review branch (eg: `semantic-pull/YYYY-MM-DD-upstream-main`) and merge those
changes into the branch of choice only after another explicit user approval.


Present the plan: third, present the following plan for how you would apply the changes for the selected update option and get user approval.
1. If no changes are needed, stop here. Continue only if changes are needed.
2. Create a new, separate review branch (eg: `semantic-pull/YYYY-MM-DD-upstream-main`). If the suggested branch name already exists, choose a unique related name and tell me what you used.
3. Apply changes according to the approved update option.
4. In general, keep stable IDs intact when the same concept remains. Add new stable IDs only for genuinely new referable sections.
5. Preserve repository meta instructions, agent steering references, licenses,
   and local fallback references unless they are obsolete and I explicitly
   approve their removal.

Present after-change summary: fourth, after applying approved changes on the review branch, summarize again:
- new tenets added
- existing tenets updated
- conflicts left for my decision
- local choices preserved
- files changed
- the review branch name
- the command I can run to inspect the diff
- the command I can run to pull the review-branch changes into my target branch

Provide me the option to merge the changes from the review branch to the target
branch. Do not merge or commit unless I explicitly approve that step.
````
