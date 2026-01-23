# Git Workflow

## Git Commit Policy (MANDATORY)

Before committing, ensure regression tests pass.

**Commit message format:**
```
<type>: <short description>

[optional body explaining why/what changed]
```

**RULES:**
- add Co-Authored-By tag "Co-Authored-By: Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
- Keep messages short (1-5 lines preferred)
- Types: feat, fix, refactor, chore, docs, build, test
- Don't commit .claude files(.claude/), output files or install dirs, etc
- Don't show 'All regression tests passed' in commit message


## Git Branch Convention (IMPORTANT)

- `master` should be based on upstream/master and match origin
- `test/*` should be based on upstream/master, plus commits for a single feature for submitting pull requests to upstream

Note: Attribution disabled globally via ~/.claude/settings.json.

## Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

