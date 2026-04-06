---
description: Show current QRSPI pipeline status
allowed-tools: Read, Glob, Grep
---

# QRSPI Status Report

Read the current state of the QRSPI pipeline and present a clear status report to the human.

## Gather State

1. Read `progress.md` for the progress log
2. Read `tasks.json` for task statuses
3. Check for reports in `reports/` to see which phases are complete
4. Check `git log --oneline -20` for recent QRSPI commits

## Present Status

Format the status as:

**Feature**: [name from progress.md]
**Current Phase**: [which phase is active]
**Progress**: [X of Y tasks completed, if in implementation phase]

**Phase Status**:
- Questions: [done/in-progress/pending] — reports/01-requirements.md
- Research: [done/in-progress/pending] — reports/02-exploration.md
- Design: [done/in-progress/pending] — reports/03-design.md
- Plan: [done/in-progress/pending] — reports/04-plan.md / tasks.json
- Implementation: [X/Y tasks done] — reports/review-task-*.md
- PR: [done/in-progress/pending]

**Last Activity**: [timestamp and description from progress.md]
**Next Action**: [what needs to happen next, and who needs to do it]

If there are blocking issues or open questions, highlight them prominently.
