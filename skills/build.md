# /build

Execute the implementation plan from a Linear ticket autonomously using strict TDD, then open a PR.

## When to Use

- A Linear ticket has an Execution Plan section (produced by `/plan-ticket`)
- User wants to hand off implementation entirely
- After `/plan-ticket` has been confirmed and the ticket is ready for engineering

## Arguments

`/build <linear-url-or-id>`

If no argument is given, ask the user to provide the Linear ticket URL or ID.

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Fetch the Ticket

Retrieve the Linear issue using the Linear Access method from `AGENTS.md`. Extract:

- Title, description, and the full Execution Plan section
- Acceptance checklist
- Parent issue if this is a sub-ticket

If the ticket has no Execution Plan section, stop and tell the user to run `/plan-ticket` first.

### 2. Prepare the Branch

```bash
git fetch origin
git checkout main
git pull origin main
git checkout -b <branch-name>
```

**Branch naming:** derive from the ticket ID and title — e.g. `feat/TEAM-123-short-description`. Use `fix/` for bug tickets, `chore/` for maintenance.

Confirm the branch is clean and up to date before proceeding:

```bash
git status
git log --oneline -3
```

### 3. Mark Ticket as In Progress

Update the ticket status to **In Progress** using the Linear Access method from `AGENTS.md`. If the state ID is unknown, look up the team's workflow states first.

### 4. Execute TDD Cycles

Work through each cycle in the Execution Plan in order. For every cycle, follow this sequence without skipping steps:

#### Step 1 — Write the spec / test

Read the cycle's test specification from the ticket. Write exactly the tests described — no more, no less. Do not write any implementation code yet.

Commit:
```bash
git add <test files>
git commit -m "test(<scope>): specify <behavior>"
```

#### Step 2 — Run the tests and confirm they fail

Run the test command specified in the plan (e.g. `rspec spec/...`, `go test ./...`, `jest src/...`).

- **Expected:** tests fail for the right reason (missing method, wrong return value, etc.)
- **If they pass already:** stop — the behavior already exists or the test is not asserting correctly. Investigate before proceeding.
- **If they fail for the wrong reason** (syntax error, missing dependency, etc.): fix the test setup, then re-run. Do not proceed until the failure is meaningful.

#### Step 3 — Implement the code

Write the minimum implementation needed to make the failing tests pass. Follow the constraints and patterns described in the cycle specification. Do not gold-plate — if the tests pass, stop.

Commit:
```bash
git add <implementation files>
git commit -m "feat(<scope>): implement <behavior>"
```

#### Step 4 — Run the tests and confirm they pass

Run the same test command. Then run the full suite:

```bash
# full suite — adjust to the project's test runner
rspec / go test ./... / jest / bundle exec rails test
```

- **Expected:** all tests pass, no regressions
- **If anything fails:** fix before moving to the next cycle. Do not carry a failing test forward.

#### Step 5 — Refactor

Review the implementation just written. Apply improvements if any of the following are true:

- Duplication with nearby code that can be extracted
- Naming that doesn't clearly express intent
- Logic that can be simplified without changing behavior
- Abstraction boundary that is inconsistent with surrounding code

Run the full test suite again after any refactor to confirm nothing broke. Commit if changes were made:

```bash
git add <changed files>
git commit -m "refactor(<scope>): <what was improved>"
```

Repeat Steps 1–5 for each cycle in the plan.

### 5. Final Verification

After all cycles are complete, run the full test suite and linter one final time:

```bash
# Tests
rspec / go test ./... / jest

# Lint / typecheck
rubocop / golangci-lint run / eslint . / tsc --noEmit
```

If anything fails, fix it before continuing. Do not proceed to the PR with a broken suite.

### 6. Open the PR

Call `/open-pr` to handle the PR creation. It will:

- Run the final checks
- Generate the PR description (referencing the Linear ticket)
- Push the branch and open the PR on GitHub
- Move the Linear ticket to "In Review"

## Rules

- Never skip or reorder the TDD cycle steps — spec before implementation, always
- Never implement code before the test is written and confirmed failing
- Never carry a failing test into the next cycle
- Never open a PR with a failing test suite or lint errors
- Keep commits granular — one commit per TDD step, not one commit per cycle
- Do not implement anything beyond what the tests require
- If the Execution Plan is missing or incomplete, stop and ask the user to run `/plan-ticket` first
- If a cycle's specification is ambiguous, stop and ask before writing the test
