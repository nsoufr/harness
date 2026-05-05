# /refine

Rigorously review a PRD from Linear, expose loopholes and gaps, then break it into scoped actionable tickets.

## When to Use

- User has a Linear issue containing a PRD (typically created by `/discover`) and wants it broken down before planning
- Before assigning or scheduling any work on a discovery issue
- When a PRD feels too large or under-specified to hand directly to an engineer

## Arguments

`/refine <linear-url-or-id>`

If no argument is given, ask the user to provide the Linear issue URL or ID.

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Fetch the PRD

Retrieve the Linear issue using the Linear Access method from `AGENTS.md`. Extract:

- Title and description (the full PRD body)
- Current state, assignee, team
- Any linked issues or sub-issues already attached

### 2. Audit the PRD for Loopholes

Go through the PRD section by section with adversarial intent — your job is to find what was missed, assumed, or glossed over. Use this checklist:

#### Jobs to Be Done
- [ ] Is each job written from the user's perspective, not the system's?
- [ ] Does the situation/motivation/outcome structure hold, or is it describing a feature instead of a goal?
- [ ] Are there jobs that different user types would have that aren't covered?
- [ ] Could any job be satisfied by a solution other than the one implied — is the PRD solving the job or solving one implementation of it?

#### Use Cases
- [ ] Does every use case map to at least one Job to Be Done?
- [ ] Are the actors specific enough (not just "user")?
- [ ] Is the current outcome described in concrete, observable terms — not vague ("it fails", "it's slow")?
- [ ] Are there use cases that exist in the codebase but weren't listed (check related flows)?
- [ ] Are negative/error paths covered, or only the happy path?

#### Completeness
- [ ] Is the user impact quantified, or just described qualitatively?
- [ ] Are all affected user types identified (not just the primary one)?
- [ ] Does the root cause fully explain the symptom, or does it stop one level too early?
- [ ] Are there edge cases in the affected flow that weren't considered?

#### Origin & Intentionality
- [ ] If the behavior was intentional, is there evidence for why it was acceptable then but not now?
- [ ] If it's a regression, is the commit/PR/deploy confirmed or only suspected?
- [ ] Could there be multiple independent causes?

#### Metrics
- [ ] Are the proposed metrics actually measurable with current instrumentation?
- [ ] Is there a baseline to compare against, or are we flying blind?
- [ ] Could the metric improve without the underlying problem being solved (proxy risk)?

#### Business Solution
- [ ] Is the desired outcome specific enough to know when it's achieved?
- [ ] Are there stakeholder or compliance constraints not mentioned?
- [ ] Does the solution conflict with any other known initiative or ongoing work?

#### Technical Solutions
- [ ] Are the options mutually exclusive, or could they be combined?
- [ ] Does each option account for rollback / rollout strategy?
- [ ] Are there data migration, backward-compatibility, or API contract implications?
- [ ] Were performance and security implications considered?
- [ ] Is there a hidden dependency on another team's system?

### 3. Present the Audit

List every gap found, grouped by severity:

```
### PRD Audit: [Issue Title]

#### Critical gaps (block refinement until resolved)
- [gap description — what's missing and why it matters]

#### Important gaps (should be resolved before work starts)
- [gap description]

#### Minor gaps (can be addressed during implementation)
- [gap description]

#### Assumptions made explicit
- [things the PRD implies but never states — flag for confirmation]
```

Ask the user: "Should we resolve the critical gaps before breaking this down, or proceed with what we have?"

### 4. Break Down into Sub-tickets

Once gaps are acknowledged, decompose the PRD into the smallest independently shippable units. Apply these rules:

- Each ticket should be completable by one engineer in one cycle
- Separate **discovery/spike** work from **implementation** work
- Separate **backend** from **frontend** where the dependency is one-way
- Separate **data migration** from **feature logic** — migrations are always their own ticket
- Separate **instrumentation / metrics** from **feature work** — observability ships first or in parallel
- If a technical option needs validation before committing, create a **spike ticket** instead of an implementation ticket

For each sub-ticket, produce:

```
### Ticket: [title]

**Type:** Feature | Bug | Spike | Chore | Migration
**Depends on:** [ticket titles this must follow, if any]
**Blocks:** [ticket titles that cannot start until this is done, if any]

**Summary:**
[2-3 sentences. What needs to be done and why.]

**Acceptance criteria:**
- [ ] [specific, testable criterion]
- [ ] [specific, testable criterion]

**Out of scope:**
- [anything explicitly excluded to keep this ticket focused]
```

### 5. Present the Breakdown

Show the full set of sub-tickets and a dependency graph (as a simple ordered list if a diagram isn't practical). Ask:

- "Does this breakdown look right? Any tickets to merge, split, or reorder?"

Incorporate feedback before creating anything in Linear.

### 6. Create Sub-tickets in Linear

Once the user confirms, create each sub-ticket as a Linear issue:

- Parent: link each sub-ticket to the original PRD issue
- Team: same team as the parent unless the user specifies otherwise
- Labels: carry over relevant labels from the parent; add `refined` to the parent issue
- Order: create in dependency order (blockers first)

Confirm back to the user with the list of created issue URLs.

## Rules

- Always read the full PRD before auditing — never skim
- Audit with adversarial intent: assume something important was missed
- Never skip the audit phase even if the PRD looks complete
- Gaps must be named specifically — "needs more detail" is not a gap
- Do not create sub-tickets until the user has approved the breakdown
- Sub-tickets must have acceptance criteria — a ticket without them is not ready
