# Tether Analytics repository instructions

## Commit and push policy

- After making any repository change, including documentation, configuration, or analytics-definition updates, stage the files changed for the current task, create a focused commit, and push the current branch to `origin`.
- When working on `main`, push the commit directly to `origin/main`.
- Do not stage or commit unrelated pre-existing working-tree changes.
- Before pushing, fetch `origin` and integrate any upstream changes. If `origin/main` has advanced, rebase the local work onto it before pushing.
- If an integration conflict occurs, tell the user in a progress update, then resolve the conflict autonomously with best judgment while preserving the intent of both sides. Re-run relevant checks, complete the rebase, commit if needed, and push.
- If a conflict cannot be safely resolved without a product or policy decision, stop and ask the user for direction.
