---
description: Resume the QRSPI pipeline from where it left off
allowed-tools: Read, Write, Glob, Grep, Bash
---

# Resume QRSPI Pipeline

The pipeline was interrupted or a new session started. Resume from the last checkpoint.

## Recovery Process

1. **Read progress.md** to find the last completed phase and activity
2. **Read tasks.json** to find the current task status
3. **Check reports/** to verify which phase deliverables exist
4. **Check git log** for the latest QRSPI commits

## Determine Resume Point

Based on the state:

- If `reports/01-requirements.md` exists but not `02-exploration.md`: Resume at Phase 2 (Research)
- If `02-exploration.md` exists but not `03-design.md`: Resume at Phase 3 (Design)
- If `03-design.md` exists but not `tasks.json` or it's empty: Resume at Phase 4 (Plan)
- If `tasks.json` has tasks with status `pending` or `in_review`: Resume at Phase 5 (Implementation)
- If all tasks are `approved`: Resume at Phase 6 (PR Creation)

## Resume

Present the current state to the human:
- "Pipeline was at Phase N: [Phase Name]. Last activity: [description]. Ready to continue?"

Wait for human confirmation before proceeding. Then follow the same pipeline rules as `/qrspi` from the appropriate phase.

**IMPORTANT**: Run the startup ritual (read progress, check git, run tests) before doing any work.
