# Engineering Rules

Always-on rules that apply to every interaction. These define the baseline engineering standards.

## Code Quality

- Write code that is correct, not clever. Prefer readability over brevity.
- Never add comments unless asked. Code should be self-documenting.
- Follow existing patterns in the codebase. Consistency > personal preference.
- Use existing libraries and utilities before adding new dependencies.
- Never assume a library is available — check package.json, Cargo.toml, etc. first.

## Testing

- Write tests for new features. Match the existing test style and framework.
- Before opening a PR, run the full test suite locally.
- Tests should verify behavior, not implementation.
- Fix tests if they break. Never skip or `xit` a failing test to make CI pass.

## Security

- Never commit secrets, API keys, or credentials.
- Never log sensitive data (passwords, tokens, PII).
- Validate and sanitize all user inputs.
- Use parameterized queries — never string-interpolate into SQL.

## Git Hygiene

- Keep commits focused and atomic. One logical change per commit.
- Write commit messages in imperative mood: "add feature" not "added feature".
- Reference Linear tickets in commits and PRs.
- Never commit generated files, `.env`, or build artifacts.
- Rebase, don't merge, when pulling in upstream changes (if team convention).
- Never force-push to shared branches without explicit permission.

## Review Readiness

Before asking for review, verify:
- [ ] All tests pass
- [ ] Lint and typecheck pass
- [ ] No stray debug output
- [ ] No commented-out code
- [ ] Changes are focused and relevant
- [ ] PR description is clear and complete
