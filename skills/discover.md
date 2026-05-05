# /discover

Analyze a problem deeply and produce a structured PRD, then publish it as a Linear project.

## When to Use

- User describes a bug, pain point, or product gap and wants to formalize it before any solution work begins
- Before writing a ticket for a non-trivial problem
- When the scope or root cause of a problem is unclear and needs structured investigation

## Arguments

`/discover <problem description>`

If no argument is given, ask the user to describe the problem before continuing.

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Clarify the Problem

Read the argument and identify whether you have enough to proceed. Ask focused clarifying questions if critical context is missing — but do not ask more than three questions at once. Key things to resolve upfront:

- Which app / surface is affected? (Rails API, Go service, Next.js frontend — check which exist in the monorepo)
- Is this already reported somewhere? (Linear issue, Slack thread, error tracker)
- Is there a known timeframe when it started?

### 2. Investigate the Codebase

Use the codebase to gather evidence. Work through this checklist:

1. **Locate the affected area** — grep for keywords from the problem description across the relevant app (`app/`, `internal/`, `src/`, etc.)
2. **Read the relevant code** — understand what the current behavior is and why it exists
3. **Check git history** — `git log --oneline --all -S "keyword"` to find when the behavior was introduced and by whom
4. **Look for tests** — do tests describe the expected behavior? Do they currently pass?
5. **Check for feature flags or config** — was this behavior gated or intentional at some point?
6. **Look for existing issues** — search Linear (via MCP or API) for related tickets that may overlap

### 3. Draft the PRD

Produce a structured document using the template below. Fill every section — write "Unknown — needs investigation" if genuinely missing rather than skipping.

---

```
## Problem: [short title]

### Context
[1-3 paragraphs. What is the system/feature involved? What is the expected behavior vs. what is actually happening? Include any relevant technical background.]

### User Impact
[Who is affected — which user types, how many (if known), how frequently? What does this cause them to experience: errors, incorrect data, degraded performance, blocked workflow? Quote any user reports or support tickets if available.]

### Jobs to Be Done
[What are users trying to accomplish when they hit this problem? Write each job in the form: "When I [situation], I want to [motivation], so I can [outcome]." Focus on the underlying goal, not the feature or UI.]

### Use Cases
[Enumerate the concrete scenarios where this problem surfaces. For each:]
- **Use case N: [name]**
  - Actor: [who — user type, role, or system]
  - Situation: [what they are doing when the problem occurs]
  - Current outcome: [what actually happens]
  - Expected outcome: [what should happen]

### Root Cause
[Why does this happen technically? Walk through the code path. Be specific — name files, functions, data flows. If the root cause is unknown, state what is known and what remains to be confirmed.]

### Origin
[Was this behavior introduced intentionally or is it a regression? If intentional: what was the original rationale, what has changed since then that makes it a problem now? If a regression: reference the commit, PR, or deploy that introduced it.]

### Metrics
[Are there any quantitative signals — error rates, latency p99, drop in conversion, support ticket volume, log counts? List what exists and what should be instrumented if currently absent.]

### Business Solution
[What outcome do we want? Frame this in user/product terms — not technical implementation. What does "fixed" look like from the user's perspective? What is the priority and why?]

### Technical Solutions
[List 2-3 concrete implementation options. For each:]
- **Option N: [name]**
  - What changes: [files / services / data]
  - Tradeoffs: [complexity, risk, reversibility]
  - Estimated effort: [S / M / L]
```

---

### 4. Ask for Review

Present the drafted PRD to the user and ask:
- "Does this capture the problem correctly? Anything missing or wrong?"

Incorporate feedback before publishing.

### 5. Publish to Linear

Once the user confirms, create a Linear project:

- **Name:** `[Discovery] <short problem title>`
- **Description:** the full PRD markdown
- **Status:** `planned` (or the appropriate initial status for the team's workflow)
- **Team:** ask the user which team to assign to if not obvious from context

Use the Linear Access method from `AGENTS.md`. Confirm the created project URL back to the user.

## Rules

- Investigate before drafting — never write the PRD from the problem description alone
- Every section must be filled or explicitly marked unknown
- Solutions go in the last two sections only — keep root cause analysis free of solution bias
- Do not create the Linear project until the user has approved the PRD
- Match the affected app's conventions when referencing code (Rails MVC paths, Go package paths, Next.js route structure)
