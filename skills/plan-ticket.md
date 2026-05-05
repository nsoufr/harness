# /plan-ticket

Analyse a Linear ticket, explore the codebase, and produce a strict TDD execution plan — delivered as an enriched ticket description and step-by-step sub-issues in Linear. This skill specifies how to execute; it does not write code.

## When to Use

- User provides a Linear ticket URL or ID (e.g. `TEAM-123`)
- Before implementing any non-trivial feature or bugfix
- After `/refine` has produced scoped sub-tickets ready for engineering

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Fetch Ticket Details

Fetch the ticket using the Linear Access method from `AGENTS.md`.

**If no ticket:** ask the user to describe the task, then proceed from Step 3.

### 2. Mark Ticket as In Progress

Update the ticket status to **In Progress** using the Linear Access method from `AGENTS.md`. If the "In Progress" state ID is unknown, first look up the team's workflow states.

### 3. Understand the Requirement

Parse the ticket to identify:

- **What** needs to be built or fixed
- **Why** it matters (context from description)
- **Where** in the codebase this lives (if obvious from title/description)
- **Acceptance criteria** — what defines "done"

If the ticket is ambiguous, ask clarifying questions before proceeding.

### 4. Explore the Codebase

Work through this checklist in order:

1. **Identify the tech stack** — this is typically a monorepo with one or more of: Rails (`Gemfile`, `app/`), Go (`go.mod`), Next.js (`package.json`, `app/` or `pages/`). Check which apps are present and scope your exploration to the relevant one(s). Note the framework, test runner, and lint tool.
2. **Read recent history** — `git log --oneline -20` to understand what has changed recently in the affected area.
3. **Find the entry point** — grep for keywords from the ticket. Start from test files, not source: tests describe expected behavior.
4. **Read existing tests first** — tests describe behavior contracts; reading them before the implementation gives you the "what" before the "how".
5. **Find similar implementations** — look for existing patterns to follow. Consistency > clever.
6. **Check dependencies** — note anything the affected module depends on that could be impacted.

### 5. Produce the Execution Plan

Draft the execution plan as a sequence of TDD cycles. Each cycle covers one coherent behavior unit (a method, endpoint, UI state, etc.) and follows this strict order:

```
## Execution Plan: [Ticket Title]

### Summary
[1-2 sentences describing the change and its scope.]

### Tech context
- Test runner: [RSpec / Go test / Jest / etc.]
- Relevant files: [list the files that will be touched]
- Patterns to follow: [existing test/implementation patterns to mirror]

### TDD Cycles

#### Cycle N: [behavior being specified]

**Step 1 — Write the spec / test**
[Describe exactly what to write: which test file, which describe/context block, which test cases to add. Name the specific behaviors to cover — including edge cases and failure paths. Do not write the code; describe what the test must assert.]

**Step 2 — Run the test and confirm it fails**
[State the exact command to run (e.g. `rspec spec/models/foo_spec.rb`, `go test ./internal/foo/...`, `jest src/foo.test.ts`). The expected outcome is a failing test for the right reason — a missing method, unexpected return value, etc. If it fails for the wrong reason, stop and investigate before proceeding.]

**Step 3 — Implement the code**
[Describe what to implement: which file, which class/module/function, what the logic should do. Reference the acceptance criteria and any patterns found in Step 4. Do not write the code; describe the intended behavior and constraints.]

**Step 4 — Run the test and confirm it passes**
[Repeat the same test command. Expected outcome: all specified tests pass, no regressions in the wider suite (run `rspec`, `go test ./...`, `jest` without file filter). If anything fails, return to Step 3.]

**Step 5 — Refactor**
[Ask the programmer: "Now that the tests pass, is there anything in the implementation worth simplifying or improving?" Prompt them to consider: duplication, naming clarity, abstraction boundaries, performance, and consistency with surrounding code. No behavior changes — tests must still pass after any refactor.]

---
[Repeat for each cycle]

### Edge Cases and Risks
- [Gotcha or constraint — and which cycle it belongs to]

### Acceptance Checklist
- [ ] [Criterion from ticket]
- [ ] All new specs/tests written before implementation
- [ ] Each test confirmed failing before implementation
- [ ] Full test suite passes after all cycles
- [ ] Lint and typecheck pass
- [ ] Refactor step completed for each cycle
```

### 6. Enrich the Ticket and Create Sub-issues

Once the plan is drafted:

**Update the ticket description** in Linear — preserve the original content at the top, then append the full execution plan as the last section:

```
---
## Execution Plan

### Summary
...

### Tech Context
...

### TDD Cycles
[all cycles, Steps 1–5 each]

### Edge Cases and Risks
...

### Acceptance Checklist
...
```

Use the Linear Access method from `AGENTS.md`. Confirm back to the user with the updated ticket URL.

### 7. Confirm with User

Before the user begins implementation, ask:
- "Does this plan look right? Any cycles to split, merge, or reorder?"

## Rules

- Always explore the codebase before producing a plan — never guess
- Read test files before source files — tests describe behavior
- Every cycle must follow the exact TDD order: spec → fail → implement → pass → refactor
- Never describe implementation before the test step — solution bias contaminates spec writing
- The refactor step is not optional — prompt it explicitly after every cycle
- This skill produces specifications, not code — never write implementation or test code
- If the ticket is ambiguous, ask clarifying questions rather than assuming
- Match the existing test framework and patterns found in the codebase
