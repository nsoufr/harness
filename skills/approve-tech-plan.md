# /approve-tech-plan

Approve a completed technical design, materialise its milestones and work items as Linear issues, and move the project into active development.

## When to Use

- After `/tech-plan` has produced a technical design document inside a Linear issue
- The document status is **Ready for Implementation** (not Draft or Open Questions)
- A human has reviewed and signed off on the design — this skill enacts the approval, it does not perform the review

## Arguments

`/approve-tech-plan <linear-url-or-id>`

If no argument is given, ask the user to provide the Linear issue URL or ID.

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Fetch and Validate the Tech Plan

Retrieve the Linear issue using the Linear Access method from `AGENTS.md`. Extract:

- The full technical design document
- The **Milestones** section at the end of the document
- Current issue status

**Hard stops — do not proceed if:**
- The document has no Milestones section → tell the user to complete `/tech-plan` first
- The document status is **Draft** → ask the user to finish the design before approving
- The document status is **Open Questions** → list the unresolved questions and tell the user they must be resolved before approval

### 2. Present the Approval Summary

Show the user what is about to be created before touching Linear:

```
## Approval Summary: [Issue Title]

### What will be created

**Milestone 1 — [name]**
Goal: [goal]
Tickets to create:
  - [work item title]
  - [work item title]

**Milestone 2 — [name]**
...

### Linear structure
- Each milestone → 1 parent issue labeled `Milestone`
- Each work item → 1 sub-issue linked to its milestone parent
- Parent issue (this tech plan) → status set to `In Progress`
- Milestone issues → created in order, labeled with their sequence

Total: [N] milestones, [M] issues
```

Ask: "Does this look right? Confirm to create everything in Linear, or tell me what to adjust."

Do not create anything until the user explicitly confirms.

### 3. Update the Parent Issue

Once confirmed, update the tech plan issue:

- Set status to **In Progress** using the Linear Access method from `AGENTS.md`
- Add a comment:
  ```
  Approved. Milestones and work items created — see sub-issues.
  ```

### 4. Create Milestones as Parent Issues

For each milestone in the document, create a Linear issue:

- **Title:** `[M{N}] [milestone name]` — e.g. `[M1] Infrastructure & Observability`
- **Description:**
  ```
  ## Milestone N: [name]

  **Goal:** [goal from tech plan]
  **Definition of done:** [definition from tech plan]
  **Dependencies:** [dependencies from tech plan]
  **Risk:** [risk from tech plan]
  ```
- **Label:** `Milestone`
- **Parent:** link to the tech plan issue
- **Status:** `Backlog`

Create milestones in sequence (M1 first). Record each created issue ID for the next step.

### 5. Create Work Item Issues

For each work item within a milestone, create a Linear sub-issue:

- **Title:** derived from the work item description — concise, imperative, specific
- **Description:**
  ```
  Part of **[M{N}] [milestone name]**.

  [Expand the work item into 2-3 sentences of context drawn from the technical design — architecture section, data design, or pattern decision that is most relevant to this item.]

  **Acceptance criteria:**
  - [ ] [derive from milestone definition of done and technical design]
  ```
- **Parent:** the milestone issue created in Step 4
- **Status:** `Backlog`
- **Label:** carry over any relevant labels from the tech plan issue (e.g. `backend`, `frontend`, `migration`, `spike`)

**Label rules:**
- Work items that are investigations or unknowns → label `Spike`
- Data migration work items → label `Migration`
- Observability/metrics work items → label `Observability`
- First milestone work items → no special sequencing label needed; milestone order is the signal

### 6. Set Milestone Dependencies

For each milestone that declares dependencies, link the blocking relationship in Linear using the appropriate method:

- MCP: use `linear_create_issue_relation` with type `blocks`
- API: use the `IssueRelation` mutation with `type: "blocks"`

This makes the dependency graph visible in Linear without requiring manual tracking.

### 7. Confirm

Report back to the user:

```
## Done

Tech plan approved and project structure created in Linear.

| Milestone | Issues created | Linear link |
|-----------|---------------|-------------|
| M1: [name] | [N] | [url] |
| M2: [name] | [N] | [url] |

Parent issue moved to In Progress: [url]

Next step: run `/plan-ticket` on any issue in M1 to produce a TDD execution plan.
```

## Rules

- Never create anything in Linear before the user has confirmed the approval summary
- Never approve a document in Draft or Open Questions status — surface the blockers instead
- Milestone issues are parents; work item issues are their children — never flatten this structure
- Work item titles must be imperative and specific — "add index to users.email" not "database work"
- Acceptance criteria must be derived from the tech plan, not invented — if they can't be derived, flag the gap
- Do not re-run `/tech-plan` — this skill enacts an already-completed design
