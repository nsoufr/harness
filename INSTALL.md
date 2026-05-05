# Installing Harness on an Existing Repository

This guide is written for agents. Follow each step in order. Do not copy or overwrite any file without completing the discovery and conflict-resolution phases first.

---

## Phase 1 — Discover the Target Repository

Read the existing state of the repository before touching anything.

### 1.1 Locate existing agent configuration files

Search for all files that may already configure an AI agent or define rules and skills:

```bash
# Agent instruction files
find . -maxdepth 3 -name "AGENTS.md" -o -name "CLAUDE.md" -o -name ".cursorrules" \
  -o -name ".windsurfrules" -o -name "copilot-instructions.md" \
  -o -name "*.mdc" -o -name ".aider*" | sort

# Existing skills or slash commands
find . -maxdepth 4 -type d -name "skills" -o -type d -name "commands" -o -type d -name "prompts" | sort

# Existing rules directories
find . -maxdepth 4 -type d -name "rules" | sort
```

Read every file found. Do not skip any.

### 1.2 Detect the target agent

Identify which agent(s) are in use:

| Signal | Agent |
|--------|-------|
| `CLAUDE.md` present | Claude Code |
| `.claude/settings.json` present | Claude Code |
| `.cursor/` directory present | Cursor |
| `.windsurfrules` present | Windsurf |
| `.github/copilot-instructions.md` present | GitHub Copilot |
| `AGENTS.md` only | OpenCode or agent-agnostic |

Note all detected agents — there may be more than one.

### 1.3 Read the existing rules and skills

For each file found in 1.1, extract:

- **Rules:** any always-on constraints or guidelines (code quality, testing, security, etc.)
- **Skills:** any slash commands, procedures, or workflow definitions
- **Process:** any documented development workflow or pipeline

Summarise what you found:

```
### Existing configuration summary

Agent(s) detected: [list]

Rules found:
- [rule topic] in [file path]

Skills found:
- [skill name] in [file path]

Process documented: yes / no
  [brief description if yes]
```

Present this summary to the user and ask: "Does this look complete? Is there anything else I should read before proceeding?"

---

## Phase 2 — Audit for Conflicts

Compare the harness content against what already exists. Work through each category.

### 2.1 Rules conflicts

For each rule topic in the harness (`AGENTS.md` — Code Quality, Testing, Security, Git Hygiene, Performance, Dependencies), check whether the existing repo has a rule on the same topic.

Classify each as:

- **Compatible** — same intent, no contradiction (safe to merge)
- **Additive** — harness adds something the repo doesn't have (safe to add)
- **Contradictory** — the two rules express different or opposite guidance

For every **contradictory** pair, present it to the user:

```
### Rules conflict: [topic]

Existing rule (from [file]):
> [exact text]

Harness rule:
> [exact text]

Options:
  A) Keep existing rule
  B) Use harness rule
  C) Merge both (describe how)
  D) Write a custom rule now

Which do you prefer?
```

Wait for the user's answer before proceeding. Do not assume.

### 2.2 Skills conflicts

For each skill in the harness, check whether a skill with the same name or same purpose already exists.

- **Same name, same purpose** — safe to replace (confirm with user first)
- **Same name, different purpose** — conflict; ask user to rename one
- **Different name, overlapping purpose** — present both and ask whether to keep one, both, or merge

```
### Skills conflict: [skill name]

Existing skill (from [file]):
> [brief description of what it does]

Harness skill:
> [brief description of what it does]

Options:
  A) Keep existing skill
  B) Use harness skill
  C) Keep both under different names — specify names
  D) Merge them — I will draft a merged version for your review

Which do you prefer?
```

### 2.3 Development process conflicts

If the existing repo documents a development process or workflow, compare it to the harness pipeline (Discover → Review PRD → Refine → Tech Plan → Approve → Plan → Build).

Present any structural differences and ask:
- "The existing process has [X]. The harness pipeline has [Y]. Should I keep the existing process, replace it with the harness pipeline, or blend them?"

---

## Phase 3 — Install

Only begin installation after all conflicts from Phase 2 have been resolved.

### 3.1 Install shared files

Copy the harness files that are agent-agnostic:

```bash
# Skills
cp -r <harness>/skills/ ./skills/

# Rules
cp -r <harness>/rules/ ./rules/

# AGENTS.md — see merge note below
```

**AGENTS.md merge rule:** Do not overwrite an existing `AGENTS.md`. Instead:
1. Read the existing file
2. Append the harness sections that are not already present (using the conflict resolutions from Phase 2)
3. Write the merged result
4. Show the diff to the user before saving

### 3.2 Install agent bridge files

For each detected agent, install the corresponding bridge:

**Claude Code**
```bash
cp <harness>/bridges/claude/CLAUDE.md ./CLAUDE.md     # or merge — see below
cp -r <harness>/bridges/claude/.claude ./.claude
```

`CLAUDE.md` merge rule: if an existing `CLAUDE.md` is present, append the harness `@`-includes that are not already referenced. Never remove existing `@`-includes.

**Cursor**
```bash
cp -r <harness>/bridges/cursor/.cursor ./.cursor
```

If `.cursor/rules/` already has `.mdc` files, do not overwrite them. Add the harness rules as additional files with distinct names.

**Windsurf**
```bash
cp <harness>/bridges/windsurf/.windsurfrules ./.windsurfrules   # merge if exists
```

**GitHub Copilot**
```bash
mkdir -p .github
cp <harness>/bridges/copilot/copilot-instructions.md .github/copilot-instructions.md  # merge if exists
```

For any bridge file that already exists: read it, merge the harness content using the same rule/skill conflict resolutions from Phase 2, show the diff to the user, and only write after confirmation.

### 3.3 Update the folder structure tree in AGENTS.md

After installation, update the `## Folder Structure` section in `AGENTS.md` to reflect the actual files present in the repository (some harness skills or rules may have been excluded per the user's conflict resolutions).

### 3.4 Verify

Run a final check to confirm everything landed correctly:

```bash
# Confirm key files exist
ls AGENTS.md skills/ rules/

# Confirm no original files were deleted
git status
```

Show the `git status` output to the user. If any deletions appear that were not explicitly approved in Phase 2, restore them immediately:

```bash
git checkout -- <file>
```

---

## Phase 4 — Confirm

Present a final installation report:

```
## Installation complete

### Files added
- [file path]

### Files merged
- [file path] — [brief description of what was merged]

### Conflicts resolved
- [topic]: [resolution chosen]

### Skipped (user chose to keep existing)
- [file or rule]

### Next step
Run `/onboard` to generate a project-level AGENTS.md tailored to this codebase,
or jump straight into the pipeline with `/discover <problem statement>`.
```

---

## Rules

- Never delete or overwrite any file without explicit user confirmation
- Never resolve a conflict by assumption — always ask
- Present one conflict at a time; do not batch them into a single question
- Always show a diff before writing a merged file
- If the user skips a conflict resolution, do not proceed with that file — ask again
- The goal is additive installation: the repo ends up with more capability, nothing lost
