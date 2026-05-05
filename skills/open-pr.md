	 # /open-pr

Verify all checks pass, then open a pull request on GitHub with a well-structured description.

## When to Use

- After completing implementation of a feature or bugfix
- When the user asks to open a PR
- Before merging any branch into main/master

## Procedure

### 1. Sanity Check

Before running any automated checks, do a quick manual review:

- Any stray `console.log`, `debugger`, or commented-out code?
- Any unstaged or uncommitted changes?

```bash
git status
git diff --stat HEAD
```

### 2. Run Tests

Execute the project's test suite:

```bash
# Auto-detect the test command (check package.json, Makefile, etc.)
npm test
npm run test
cargo test
pytest
make test
```

**If tests fail:**
- Report failures clearly
- Fix the failures before proceeding
- Do NOT open a PR with failing tests

**If no tests exist:**
- Note it in the PR description
- Ask the user if they want tests added before opening the PR

### 3. Run Lint and Typecheck

```bash
npm run lint
npm run typecheck
cargo clippy
ruff check .
```

Fix any issues before proceeding.

### 4. Rebase and Template Check

```bash
# Check if branch is behind main
git fetch origin
git log HEAD..origin/main --oneline
```

If behind: warn the user and offer to rebase before pushing. Do not silently rebase — rebasing rewrites history.

```bash
# Check for PR template
ls .github/pull_request_template.md 2>/dev/null
```

If a PR template exists, use it as the structure for the PR description in the next step.

### 5. Generate PR Description

Use `git log` to review commits on the branch and draft a description:

```markdown
## Summary
[1-3 bullet points describing the changes and why]

## Changes
- [Specific change 1]
- [Specific change 2]

## Testing
- [ ] All tests pass
- [ ] Lint passes
- [ ] Manual verification: [describe what was manually tested]

## Screenshots / Demos
[if UI changes, include screenshots or asciinema]

Closes [TEAM-123](https://linear.app/...)
```

### 6. Create the PR

```bash
# Push branch if not already pushed
git push -u origin HEAD

# Create PR with gh CLI
gh pr create \
  --title "type: concise description" \
  --body "$(cat <<'EOF'
...description from step 5...
EOF
)"
```

### 7. Update Linear Ticket (if MCP configured)

If the Linear MCP server is available, move the ticket to "In Review":

```
linear_update_issue(id: "TEAM-123", stateId: "<in-review-state-id>")
```

### 8. Report

Output the PR URL and a summary:

```
PR opened: https://github.com/owner/repo/pull/42
Title: fix: handle null edge case in user profile
Branch: fix/user-profile-null → main
Tests: ✓ passing
Lint: ✓ passing
```

## PR Title Convention

Follow the repo's existing convention. If none is detected, use:

```
type: concise description
```

Where `type` is one of: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`

## Rules

- NEVER open a PR with failing tests
- NEVER push secrets or `.env` files
- NEVER force-push to `main` or `master`
- NEVER silently rebase — always confirm with the user first
- Always include the Linear ticket reference if one exists
- Only commit what's necessary — no unrelated changes
- Check for a PR template before writing the description
