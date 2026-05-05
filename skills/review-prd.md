# /review-prd

Guide a reviewer through a Linear PRD — build context, surface what needs scrutiny, gather information from the codebase, and post structured review comments back to Linear.

## When to Use

- A team member needs to review a PRD but lacks the product or technical context to do it well
- Before a refinement session or planning meeting, to make review time more effective
- When a PRD touches unfamiliar parts of the codebase and the reviewer needs grounding

## Arguments

`/review-prd <linear-url-or-id>`

If no argument is given, ask the user to provide the Linear issue URL or ID.

> For all Linear operations follow the **Linear Access** priority order defined in `AGENTS.md`.

## Procedure

### 1. Fetch the PRD

Retrieve the Linear issue using the Linear Access method from `AGENTS.md`. Extract the full description, current state, assignee, team, and any linked issues.

### 2. Orient the Reviewer

Before diving into details, give the reviewer a one-screen briefing:

```
### PRD Briefing: [Issue Title]

**What this is about:**
[1-2 sentences. Plain language summary of the problem and proposed solution.]

**Why it matters:**
[Who is affected and what is the consequence of not solving it.]

**Scope:**
[What is explicitly in scope. What is explicitly out of scope.]

**Key decisions this PRD asks you to validate:**
- [Decision or assumption 1]
- [Decision or assumption 2]

**Estimated reading time:** ~[N] minutes
```

Ask: "Does this match your understanding, or is there context I'm missing before we go deeper?"

### 3. Build Codebase Context

Investigate the parts of the codebase the PRD touches to give the reviewer grounding:

1. **Locate affected code** — find the files, modules, or services mentioned or implied by the PRD
2. **Summarize current behavior** — describe what the code does today in plain terms, referencing specific file paths
3. **Identify technical constraints** — note anything in the current implementation that the PRD may have missed or that constrains the proposed solutions
4. **Surface related history** — `git log --oneline -S "keyword"` to find relevant past changes; flag any that add context to the problem origin
5. **Check test coverage** — does the affected area have tests? Do they reflect the behavior the PRD describes?

Present findings as:

```
### Codebase Context

**Affected area:** [file paths / service names]

**How it works today:**
[Plain-language description. Reference specific files/functions.]

**Constraints the PRD should account for:**
- [constraint — and where in the code it lives]

**Relevant history:**
- [commit or PR reference — what changed and when]

**Test coverage:** [exists / partial / missing — and what's covered]
```

### 4. Guide the Review

Walk the reviewer through the PRD section by section with prompts that focus attention on what matters most:

#### Problem & Context
- Does the description match what you see in the codebase?
- Is the scope drawn in the right place — too wide, too narrow, or just right?

#### Jobs to Be Done
- Do these jobs reflect what real users actually need, or are they retrofitted to justify a solution?
- Are there user types whose jobs are missing?

#### Use Cases
- Are there use cases you know about from support, ops, or your own experience that aren't listed?
- Do the current/expected outcome pairs match what you observe in production?

#### Root Cause
- Does the technical explanation hold up against what you know of the code?
- Is this the real root cause, or a symptom of something deeper?

#### Metrics
- Are these the right things to measure?
- Do we have the instrumentation to measure them today?

#### Business Solution
- Is this the right outcome to optimize for?
- Does it conflict with anything the team is currently working on?

#### Technical Solutions
- Are the options realistic given the current codebase?
- Is there an obvious option that wasn't considered?
- Which option would you pick, and why?

For each section, ask the reviewer: "Any concerns here, or does this look solid?"

### 5. Collect Reviewer Input

After guiding through each section, ask the reviewer to rate each section:

- **Approved** — looks good, no concerns
- **Needs clarification** — a question or ambiguity to resolve before work starts
- **Blocking concern** — something that must be addressed before this PRD can be refined or planned

Record the reviewer's input for the next step.

### 6. Post Comments to Linear

Once the review is complete, post the findings as comments on the Linear issue using the Linear Access method from `AGENTS.md`.

Structure the comments as follows:

**One top-level comment summarising the review:**
```
## PRD Review — [Reviewer name / date]

**Overall verdict:** Approved / Approved with concerns / Blocked

**Summary:**
[2-3 sentences on the overall quality and readiness of the PRD.]

**Sections reviewed:** Context · JTBD · Use Cases · Root Cause · Metrics · Business Solution · Technical Solutions
```

**One comment per blocking concern or clarification needed**, so each can be resolved independently:
```
**[Section name] — [Needs clarification | Blocking concern]**

[Specific question or issue. Reference the relevant part of the PRD directly.]

Suggested action: [what the author should do to resolve this]
```

Confirm back to the user with the Linear issue URL and a count of comments posted.

## Rules

- Always orient the reviewer before jumping into detail — context first, critique second
- Build codebase context before guiding the review — never ask the reviewer to validate something you haven't verified yourself
- Keep guidance prompts open-ended — do not lead the reviewer toward a conclusion
- One Linear comment per concern — do not bundle multiple issues into one comment
- Do not post comments until the reviewer has finished and confirmed the findings
- If the PRD is missing major sections (Jobs to Be Done, Use Cases, Root Cause), flag this before starting the guided review and ask whether to proceed or ask the author to fill gaps first
