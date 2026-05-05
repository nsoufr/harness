# Harness

A collection of AI agent skills and always-on rules that turn any AI coding agent into a structured engineering teammate — from problem discovery through to a merged PR.

## What it is

Harness gives your agent a shared vocabulary for how software is built. It defines a pipeline of slash-command skills, each with a clear input, output, and procedure. Rather than prompting the agent differently every time, you drop Harness into your project and everyone on the team gets the same consistent behaviour.

It works with Claude Code, Cursor, Windsurf, GitHub Copilot, and any agent that reads `AGENTS.md`.

## The pipeline

Each stage has a dedicated skill. You can invoke any skill directly as long as you have the required input artifact.

| Stage | Skill | Input | Output |
|-------|-------|-------|--------|
| 1. Discover | `/discover` | Problem statement | Linear issue with PRD |
| 2. Review PRD | `/review-prd` | Linear PRD link | Review comments posted to Linear |
| 3. Refine | `/refine` | Linear PRD link | Audited PRD; sub-tickets in Linear |
| 4. Tech Plan | `/tech-plan` | Linear PRD link | Technical design document in Linear |
| 5. Approve | `/approve-tech-plan` | Linear tech plan link | Milestones and work items in Linear |
| 6. Plan | `/plan-ticket` | Linear ticket link | TDD execution plan written into ticket |
| 7. Build | `/build` | Linear ticket link | Branch with TDD commits + open PR |

### What each skill does

**`/discover`** — Takes a problem statement, investigates the codebase and existing Linear issues, and produces a structured PRD covering: context, jobs to be done, use cases, root cause, metrics, business solution, and technical options. Publishes the PRD as a Linear issue.

**`/review-prd`** — Guides a reviewer through a PRD they didn't write. Builds codebase context first, then walks through each section with focused questions, collects verdicts (approved / needs clarification / blocking), and posts one Linear comment per concern.

**`/refine`** — Audits a PRD adversarially for gaps across JTBD, use cases, root cause, metrics, business solution, and technical options. Breaks the problem into independently shippable sub-tickets, each with acceptance criteria and explicit dependencies.

**`/tech-plan`** — Produces a technical design document covering architecture (C4 diagrams in Mermaid), data model, API design, patterns and decisions, security, performance, observability, cost and capacity, and a milestone breakdown. Stops and generates an open-questions document if the PRD has too many unknowns to design against.

**`/approve-tech-plan`** — Enacts a confirmed tech plan. Creates milestone parent issues and work item sub-issues in Linear, wires dependency relationships, and moves the parent issue to In Progress. Shows an approval summary before creating anything.

**`/plan-ticket`** — Explores the codebase, then produces a strict TDD execution plan written directly into the Linear ticket: one cycle per behavior unit, each cycle with spec → confirm failure → implement → confirm passing → refactor steps.

**`/build`** — Executes the execution plan from a ticket autonomously. Checks out from latest main, follows the TDD cycles commit by commit, and calls `/open-pr` when all cycles pass.

## Always-on rules

`AGENTS.md` also defines rules that apply to every interaction without being invoked:

- **Code quality** — correct over clever, follow existing patterns, no unnecessary dependencies
- **Testing** — tests verify behavior not implementation, never skip a failing test
- **Security** — no secrets in commits, no PII in logs, parameterized queries
- **Git hygiene** — atomic commits, imperative messages, Linear ticket references
- **Performance** — no N+1s, no blocking the main thread, profile before optimizing
- **Dependencies** — check before adding, commit lockfiles, audit new packages

## Installation

### Fresh project

Copy the files for your agent:

**Claude Code**
```bash
cp -r skills/ rules/ .
cp bridges/claude/CLAUDE.md .
cp -r bridges/claude/.claude .
```

**Cursor**
```bash
cp -r skills/ rules/ .
cp -r bridges/cursor/.cursor .
```

**OpenCode / any AGENTS.md-native agent**
```bash
cp -r skills/ rules/ .
# AGENTS.md is read natively — no extra steps needed
```

**Windsurf**
```bash
cp -r skills/ rules/ .
cp bridges/windsurf/.windsurfrules .
```

**GitHub Copilot**
```bash
cp -r skills/ rules/ .
mkdir -p .github && cp bridges/copilot/copilot-instructions.md .github/
```

### Existing project

Use `INSTALL.md` — it guides the agent through discovering your existing configuration, detecting conflicts with your current rules and skills one at a time, merging what's compatible, and asking you to decide on anything contradictory. Nothing is overwritten without your confirmation.

```
# In your agent
@INSTALL.md install harness into this project
```

## Repository structure

```
harness/
├── AGENTS.md                    # Always-on rules + pipeline overview (loaded automatically)
├── INSTALL.md                   # Agent-executable installation guide for existing projects
├── skills/                      # Slash-command procedures
│   ├── discover.md
│   ├── review-prd.md
│   ├── refine.md
│   ├── tech-plan.md
│   ├── approve-tech-plan.md
│   ├── plan-ticket.md
│   ├── build.md
│   └── open-pr.md
├── rules/                       # Standalone rule files for per-file @includes
│   └── engineering-rules.md
└── bridges/                     # Agent-specific adapter files
    ├── claude/                  # Claude Code
    │   ├── CLAUDE.md
    │   └── .claude/settings.json
    └── cursor/                  # Cursor
        └── .cursor/rules/engineering.mdc
```

## Contributing

To add a skill: create a `.md` file in `skills/` and add a row to the pipeline table in `AGENTS.md`. Follow the format:

```
# /skill-name

[One sentence: what it does and why.]

## When to Use
## Arguments
## Procedure
### 1. ...
## Rules
```

To add a rule set: create a `.md` file in `rules/` using assertive constraints ("Never X", not "Try to X").

## License

MIT
