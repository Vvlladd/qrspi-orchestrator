# QRSPI Pipeline

Multi-agent orchestration for feature development with human-in-the-loop checkpoints.

## Pipeline Phases

Execute phases in order. **STOP after each phase and wait for human approval before proceeding.** Never auto-proceed.

### Phase 1: Questions

- Read the feature request
- Scan the codebase for relevant patterns, files, and conventions (max 10 file reads)
- Identify ambiguities, edge cases, unstated assumptions
- Write `reports/01-requirements.md` with: Summary, Acceptance Criteria (specific, testable), Edge Cases, Out of Scope, Open Questions, Existing Code Context
- Present summary and ask: "Approve, request changes, or reject?"

### Phase 2: Research

Explore the codebase in depth:
- Map all relevant files (models, protocols, engines, configs, tests)
- Analyze existing patterns and conventions
- Identify dependencies (internal modules, external packages, APIs)
- Assess risks and complexity
- Write `reports/02-exploration.md` with: File Map, Existing Patterns, Dependencies, Architecture Notes, Risks, Suggested Approach
- Cite file paths and line numbers for every claim
- Present summary and ask for approval

### Phase 3: Design

Produce a clear, actionable design:
- Read requirements and exploration report
- Design component boundaries, interfaces, data flow
- Define test strategy per component
- Write `reports/03-design.md` with: Overview, Components (files, responsibility, interface, dependencies), Data Flow, Interface Contracts, Error Handling, Test Strategy, Migration/Breaking Changes

Then challenge your own design:
- Is this the simplest approach? What's simpler?
- What happens at scale, under load, with concurrent users?
- What are the failure modes?
- Are interfaces over-engineered or under-specified?
- Any security concerns? (auth, injection, data exposure, input validation)

Revise the design based on self-critique. Present summary and ask for approval.

### Phase 4: Plan

Decompose the design into ordered tasks:
- Each task: 2-5 minutes of implementation work
- One concern per task (don't mix implementation with tests)
- Tests are separate tasks
- Include setup tasks (packages, migrations, configs)
- Final task is always integration verification
- Write `tasks.json` with: id, title, description, files, acceptance_criteria, depends_on, estimated_minutes, status
- Write `reports/04-plan.md` with human-readable table
- Present task breakdown and ask for approval (human can reorder, split, merge, remove)

### Phase 5: Implement + Review

For each task, in order:

**Implement:**
1. Read `progress.md`, `tasks.json`, and `reports/03-design.md`
2. Run tests to establish baseline
3. Implement the ONE task following existing codebase patterns
4. Run tests — never commit if broken
5. **BEFORE committing: present to the human exactly what changed** — list every file modified/created, summarize each change, and show the git diff summary. Ask: "These are the changes for Task N. Approve commit, request changes, or reject?"
6. **WAIT for human approval.** Do NOT commit until the human explicitly approves.
7. Commit: `[QRSPI Task N] short description`
8. Update `tasks.json` status to `in_review`
9. Update `progress.md`

**Self-Review:**
1. Review the git diff objectively
2. Check each acceptance criterion from the task
3. Run full test suite
4. Check code quality: pattern consistency, error handling, edge cases
5. Write `reports/review-task-N.md` with: Verdict (PASS/FAIL), Acceptance Criteria check, Code Quality, Test Results, Issues Found
6. If FAIL: fix issues, re-commit, re-review
7. If PASS: update `tasks.json` to `approved`

**Report to human after EVERY task.** Human can pause, redirect, or intervene.

### Phase 6: Pull Request

- Verify all tasks in `tasks.json` are `approved`
- Run full test suite one final time
- Create PR with: Summary, Changes, Design Decisions, Test Coverage, Review Checklist, QRSPI Audit Trail (links to all reports)
- Present PR for final human review

## Working Files

Create these in the project root:
- `progress.md` — Activity log (update after every phase and task)
- `tasks.json` — Structured task list with status tracking
- `reports/` — Phase deliverables and review reports

## Startup Ritual

Every session, before doing any work:
1. Read `progress.md` — what's been done
2. Read `tasks.json` — current task state
3. Read relevant `reports/` files
4. Check `git log --oneline -10` — recent changes

## Progress Reporting

After every phase and task, update `progress.md`:
```
## [Timestamp] Phase N: [Phase Name]
- Status: [completed/blocked/needs-human-input]
- Output: [deliverable file path]
- Summary: [1-2 sentences]
- Next: [what happens next, pending human approval]
```

## Critical Rules

- **NEVER skip a human gate.** Every phase transition requires explicit human approval.
- **NEVER commit without human approval.** Before every git commit, present a summary of all changes (files modified, what changed, git diff) and wait for explicit approval.
- **NEVER proceed on assumption.** If the human hasn't responded, wait.
- **Report everything.** No surprises.
- **Small tasks only.** If any task would take more than 5 minutes, split it.
- **Follow existing patterns.** Match the codebase's style, not your preferences.
- **Stop reading after 10 files per phase.** Write your output with what you have. Note gaps as Open Questions.

## Resume

If starting a new session mid-pipeline, check which reports exist to determine resume point:
- `01-requirements.md` exists but not `02-exploration.md` → Resume Phase 2
- `02-exploration.md` exists but not `03-design.md` → Resume Phase 3
- `03-design.md` exists but no `tasks.json` → Resume Phase 4
- `tasks.json` has pending/in_review tasks → Resume Phase 5
- All tasks approved → Resume Phase 6

Present current state and wait for human confirmation before continuing.
