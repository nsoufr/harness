# Harness — AI Coding Agent Skills & Rules

A reusable collection of slash-command skills and always-on rules for AI coding agents.
Copy the relevant files into any project to equip your agent with these capabilities.

---

## Always-On Rules

These rules apply to every interaction automatically.

### Software Development Process

Work follows a structured pipeline from problem discovery to shipped code. Each stage has a dedicated skill. Skills can be invoked directly as long as the required input artifact is available.

| Stage         | Skill                                               | Input                 | Output                                        |
| ------------- | --------------------------------------------------- | --------------------- | --------------------------------------------- |
| 1. Discover   | [`/discover`](skills/discover.md)                   | Problem statement     | Linear issue with PRD                         |
| 2. Review PRD | [`/review-prd`](skills/review-prd.md)               | Linear PRD link       | Structured review comments posted to Linear   |
| 3. Refine     | [`/refine`](skills/refine.md)                       | Linear PRD link       | Audited PRD; sub-tickets created in Linear    |
| 4. Tech Plan  | [`/tech-plan`](skills/tech-plan.md)                 | Linear PRD link       | Technical design document published to Linear |
| 5. Approve    | [`/approve-tech-plan`](skills/approve-tech-plan.md) | Linear tech plan link | Milestones and work items created in Linear   |
| 6. Plan       | [`/plan-ticket`](skills/plan-ticket.md)             | Linear ticket link    | Execution plan written into ticket            |
| 7. Build      | [`/build`](skills/build.md)                         | Linear ticket link    | Branch with TDD commits + open PR             |


### Linear Access

For all Linear operations, use whichever method is available — in priority order:

1. **MCP tools** (preferred): `linear_get_issue`, `linear_update_issue`, `linear_create_issue`, `linear_get_issue_states`, `linear_get_teams`, `linear_search_issues`, etc.
2. **GraphQL API via curl** (if no MCP):
   ```bash
   curl -s -X POST https://api.linear.app/graphql \
     -H "Authorization: $LINEAR_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"query": "..."}'
   ```
3. **Ask the user** (if no API key): output the content as markdown so the user can act on Linear manually.

---

### Code Quality

- Write code that is correct, not clever. Prefer readability over brevity.
- Never add comments unless asked. Code should be self-documenting.
- Follow existing patterns in the codebase. Consistency > personal preference.
- Use existing libraries and utilities before adding new dependencies.
- Never assume a library is available — check `package.json`, `Cargo.toml`, etc. first.

### Testing

- Write tests for new features. Match the existing test style and framework.
- Before opening a PR, run the full test suite locally.
- Tests should verify behavior, not implementation.
- Fix tests if they break. Never skip or `xit` a failing test to make CI pass.

### Security

- Never commit secrets, API keys, or credentials.
- Never log sensitive data (passwords, tokens, PII).
- Validate and sanitize all user inputs.
- Use parameterized queries — never string-interpolate into SQL.

### Git Hygiene

- Keep commits focused and atomic. One logical change per commit.
- Write commit messages in imperative mood: "add feature" not "added feature".
- Reference Linear tickets in commits and PRs.
- Never commit generated files, `.env`, or build artifacts.
- Rebase, don't merge, when pulling in upstream changes (if team convention).
- Never force-push to shared branches without explicit permission.

### Performance

- Prefer batched reads/writes over N+1 loops.
- Never fetch data inside a render loop or unbounded iteration.
- Don't block the main thread — move CPU-heavy work to workers or async.
- Profile before optimizing. No premature optimization.

### Dependencies

- Check for existing utilities before adding a new dependency.
- Never add a dependency used in only one place that is trivially implementable inline.
- Commit lockfiles (`package-lock.json`, `Cargo.lock`, `uv.lock`). Never omit them.
- Run `npm audit` / `cargo audit` before adding a new package.

### Review Readiness

Before asking for review, run `/open-pr` — it enforces this checklist automatically:

- [ ] All tests pass
- [ ] Lint and typecheck pass
- [ ] No stray debug output or commented-out code
- [ ] Changes are focused and relevant
- [ ] PR description is clear and complete
- [ ] Linear ticket referenced

---

## Skills (Slash Commands)

Invoke on-demand as needed:

| Command | File | What it does |
|---------|------|-------------|
| `/discover` | [skills/discover.md](skills/discover.md) | Investigate a problem, produce a structured PRD, publish to Linear |
| `/review-prd` | [skills/review-prd.md](skills/review-prd.md) | Guide a reviewer through a PRD, gather codebase context, post comments to Linear |
| `/refine` | [skills/refine.md](skills/refine.md) | Audit a PRD for gaps, break it into scoped sub-tickets in Linear |
| `/tech-plan` | [skills/tech-plan.md](skills/tech-plan.md) | Produce a technical design document covering architecture, data, security, observability, cost, and milestones |
| `/approve-tech-plan` | [skills/approve-tech-plan.md](skills/approve-tech-plan.md) | Approve a completed tech plan, create milestones and work items in Linear, move project to In Progress |
| `/plan-ticket` | [skills/plan-ticket.md](skills/plan-ticket.md) | Explore the codebase, produce a strict TDD execution plan, write it into the Linear ticket |
| `/build` | [skills/build.md](skills/build.md) | Execute a planned ticket autonomously via TDD cycles, open a PR when done |
| `/open-pr` | [skills/open-pr.md](skills/open-pr.md) | Run tests + lint, rebase check, generate a PR description, open the PR on GitHub |
| `/debug` | [skills/debug.md](skills/debug.md) | Reproduce a bug, form hypotheses, trace to root cause, apply minimal fix |
| `/review` | [skills/review.md](skills/review.md) | Review the current branch's diff against engineering rules |
| `/refactor` | [skills/refactor.md](skills/refactor.md) | Restructure code safely: tests before, tests after, no behavior changes |
| `/onboard` | [skills/onboard.md](skills/onboard.md) | Explore a new project and generate a project-level AGENTS.md |

---

## Deploy

Copy files into your project based on which agent you use.

### Claude Code

```bash
cp -r skills/ rules/ .
cp bridges/claude/CLAUDE.md .
cp -r bridges/claude/.claude .
```

### Cursor

```bash
cp -r skills/ rules/ .
cp -r bridges/cursor/.cursor .
```

### OpenCode

```bash
cp -r skills/ rules/ .
# AGENTS.md is read natively — no extra steps needed
```

### Windsurf

```bash
cp -r skills/ rules/ .
cp bridges/windsurf/.windsurfrules .
```

### GitHub Copilot

```bash
cp -r skills/ rules/ .
mkdir -p .github && cp bridges/copilot/copilot-instructions.md .github/
```

---

## Folder Structure

```
harness/
├── AGENTS.md                    # This file — always-on rules + skill index
├── rules/                       # Standalone rule files (for per-file @includes)
│   ├── engineering-rules.md
│   ├── performance-rules.md
│   └── dependency-rules.md
├── skills/                      # Skill procedures
│   ├── plan-ticket.md
│   ├── open-pr.md
│   ├── debug.md
│   ├── review.md
│   ├── refactor.md
│   └── onboard.md
└── bridges/                     # Agent-specific adapter files
    ├── claude/                  # Claude Code
    │   ├── CLAUDE.md
    │   └── .claude/
    │       └── settings.json
    ├── cursor/                  # Cursor
    │   └── .cursor/
    │       ├── rules/
    │       │   └── engineering.mdc
    │       └── commands/
    │           ├── plan-ticket.md
    │           └── open-pr.md
    ├── windsurf/                # Windsurf
    │   └── .windsurfrules
    └── copilot/                 # GitHub Copilot
        └── copilot-instructions.md
```



## Contributing

To add a new skill or rule set, create a `.md` file in the appropriate directory and link it in the tables above. Follow the existing format:
- Skills: purpose header → When to Use → numbered Procedure → Rules block
- Rules: assertive constraints ("Never X"), not suggestions ("try to X")
