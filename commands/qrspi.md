---
description: Start the QRSPI pipeline for a feature request
allowed-tools: Read, Write, Glob, Grep, Bash
argument-hint: <feature description>
---

# QRSPI Pipeline Orchestrator

You are starting the QRSPI multi-agent pipeline. This is the main entry point.

## Setup

1. Create the working directories if they don't exist:
   ```
   mkdir -p reports/
   ```

2. Initialize `progress.md` if it doesn't exist:
   ```markdown
   # QRSPI Progress Log
   Feature: $ARGUMENTS
   Started: [current timestamp]
   ```

3. Initialize `tasks.json` as empty if it doesn't exist.

## Pipeline Execution

Execute each phase in order. **STOP after each phase and wait for human approval before proceeding.**

### Phase 1: Questions
Delegate to the **clarifier** agent with the feature description: `$ARGUMENTS`

After the clarifier produces `reports/01-requirements.md`, present a summary to the human and ask:
- "Requirements are ready. Review reports/01-requirements.md and let me know: approve, request changes, or reject."

**DO NOT proceed to Phase 2 until the human explicitly approves.**

### Phase 2: Research (Agent Team)
Create an **agent team** with 2-3 explorer teammates, each assigned a different area:
- Explorer 1: Core feature area (the primary files and modules)
- Explorer 2: Adjacent systems (dependencies, integrations, APIs)
- Explorer 3: Tests and infrastructure (existing test patterns, CI, configs)

Have the explorers share findings with each other. The lead synthesizes into `reports/02-exploration.md`.

Present summary and ask for approval before proceeding.

### Phase 3: Design (Agent Team)
Create an **agent team** with 3 teammates:
- **Architect**: Produces the design document
- **Critic**: Challenges every design decision, proposes alternatives
- **Security Reviewer**: Audits for security implications

Require plan approval from the architect before they write. The critic and security reviewer message the architect with feedback. The architect revises until the team converges.

The lead writes the final `reports/03-design.md`.

Present summary and ask for approval before proceeding.

### Phase 4: Plan
Delegate to the **planner** agent to decompose the design into tasks.

Present the task breakdown and ask for approval. The human can reorder, split, merge, or remove tasks.

### Phase 5: Implement + Review (Agent Team)
Create an **agent team** with 2 teammates:
- **Builder**: Implements one task at a time
- **Reviewer**: Reviews each completed task in a fresh context

Pipeline pattern:
1. Builder picks up task N, implements, commits
2. Builder messages Reviewer that task N is ready
3. Reviewer reviews task N, writes `reports/review-task-N.md`
4. If PASS: Reviewer messages the lead, lead reports to human, Builder picks up task N+1
5. If FAIL: Reviewer messages Builder with specific feedback, Builder fixes and resubmits

**Report to the human after EVERY task review.** The human can pause, redirect, or intervene at any point.

### Phase 6: Pull Request
After all tasks are approved, delegate to the **pr-creator** agent.

Present the PR for final human review before merge.

## Progress Reporting

After every phase and every task, update `progress.md` with:
```markdown
## [Timestamp] Phase N: [Phase Name]
- Agent: [agent name]
- Status: [completed/blocked/needs-human-input]
- Output: [deliverable file path]
- Summary: [1-2 sentence summary]
- Next: [what happens next, pending human approval]
```

## Critical Rules

- **NEVER skip a human gate.** Every phase transition requires explicit human approval.
- **NEVER commit without human approval.** Before every git commit, present a full summary of changes (files modified, what changed, git diff) and wait for explicit "approve" from the human.
- **NEVER proceed on assumption.** If the human hasn't responded, wait.
- **Report everything.** The human should never be surprised by what's happening.
- **Small tasks only.** If any task is estimated at more than 5 minutes, flag it for splitting.
