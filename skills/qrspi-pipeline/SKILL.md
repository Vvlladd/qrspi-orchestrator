---
name: qrspi-pipeline
description: >
  This skill should be used when the user mentions "QRSPI", "pipeline", "multi-agent development",
  "agent orchestration", "structured development workflow", or asks to "plan and implement a feature
  with review". It provides the knowledge base for the QRSPI multi-agent orchestration system that
  decomposes software development into human-approved phases with specialized agents.
version: 0.1.0
---

# QRSPI Pipeline Orchestration

The QRSPI pipeline is a multi-agent orchestration system for software development with human-in-the-loop checkpoints at every phase.

## Quick Start

Run `/qrspi <feature description>` to start the full pipeline. Run `/qrspi-status` to check progress. Run `/qrspi-resume` to continue after a session break.

## Pipeline Phases

| Phase | Agent(s) | Execution | Human Gate |
|-------|----------|-----------|------------|
| **Q** - Questions | clarifier | Subagent (single) | Approve requirements |
| **R** - Research | explorer x2-3 | **Agent Team** (parallel) | Approve exploration report |
| **S** - Structure/Design | architect + critic + security-reviewer | **Agent Team** (parallel debate) | Approve design document |
| **P** - Plan | planner | Subagent (single) | Approve task breakdown |
| **I** - Implement | builder + reviewer | **Agent Team** (pipelined) | Approve each task review |
| **PR** - Pull Request | pr-creator | Subagent (single) | Approve and merge PR |

## Agent Team Phases (Detail)

### Research Team (Phase R)
Spawn 2-3 explorer teammates, each assigned a different area of the codebase. They explore in parallel and share findings. The lead synthesizes into a single exploration report.

Example spawn prompt:
```
Create an agent team for QRSPI Research phase. Spawn explorer teammates:
- Explorer 1: investigate the core auth module at src/auth/
- Explorer 2: investigate API layer and middleware at src/api/
- Explorer 3: investigate existing tests and CI config at tests/ and .github/
Have them share findings. Synthesize into reports/02-exploration.md.
```

### Design Team (Phase S)
Spawn 3 teammates with different agent types. The architect proposes, the critic challenges, and the security reviewer audits. Require plan approval for the architect.

Example spawn prompt:
```
Create an agent team for QRSPI Design phase. Spawn teammates:
- Use the architect agent type to design the solution. Require plan approval.
- Use the critic agent type to challenge every design decision.
- Use the security-reviewer agent type to audit for vulnerabilities.
Have them debate and converge on a design. Write final to reports/03-design.md.
```

### Build Team (Phase I)
Spawn 2 teammates: builder and reviewer. The builder implements one task at a time and the reviewer reviews in a fresh context. They pipeline: builder works on task N+1 while reviewer reviews task N.

Example spawn prompt:
```
Create an agent team for QRSPI Implementation phase. Spawn teammates:
- Use the builder agent type. Start with task 1 from tasks.json.
- Use the reviewer agent type. Wait for builder to finish task 1, then review.
After each review, report the verdict to me. Don't proceed to the next task until I approve.
```

## Memory Bridging Artifacts

These files enable continuity across context windows:

| File | Purpose | Updated By |
|------|---------|------------|
| `progress.md` | Sequential log of all activity | Every agent, every phase |
| `tasks.json` | Structured task list with status | Planner (creates), Builder (updates), Reviewer (updates) |
| `reports/*.md` | Phase deliverables and reviews | Each phase's agent(s) |
| Git history | Code changes with structured commits | Builder only |

## Startup Ritual

Every agent session must begin by reading:
1. `progress.md` — what's been done
2. `tasks.json` — current task state
3. The relevant `reports/` files for their phase
4. `git log --oneline -10` — recent code changes

This takes 10-20 seconds but prevents re-exploration and drift.

## Human Gate Protocol

After every phase output:
1. Present a concise summary of what was produced
2. Link to the output file for full review
3. Ask explicitly: "Approve to continue, request changes, or reject?"
4. **WAIT** for the human response. Never auto-proceed.

After every task review (during implementation):
1. Present the reviewer's verdict (PASS/FAIL)
2. If FAIL: present the specific issues and ask if the builder should fix them
3. If PASS: ask if the human approves moving to the next task

## Troubleshooting

- **Pipeline feels slow**: The human gates are the bottleneck by design. If you want faster flow, you can approve multiple tasks at once: "Approve tasks 3-5 without stopping between them."
- **Agent team not spawning**: Ensure `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set in settings.
- **Context exhaustion**: If a session runs out of context, use `/qrspi-resume` to pick up from the last checkpoint.
- **Task too big**: If a builder task takes more than 5 minutes, stop and ask the planner to split it.

For detailed reference on agent definitions, see the agent files in the `agents/` directory of this plugin.
