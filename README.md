# QRSPI Orchestrator

Multi-agent orchestration plugin for software development with human-in-the-loop checkpoints at every phase.

## What It Does

QRSPI decomposes feature development into 6 phases, each handled by specialized agents with mandatory human approval gates between phases:

1. **Q — Questions** — Clarifier agent produces structured requirements
2. **R — Research** — Explorer agent team maps the codebase in parallel
3. **S — Structure/Design** — Architect + Critic + Security Reviewer debate the design
4. **P — Plan** — Planner decomposes into 2-5 minute tasks
5. **I — Implement** — Builder + Reviewer pipeline (implement one, review one)
6. **PR — Pull Request** — PR agent assembles the final PR

## Agents

| Agent | Model | Phase | Purpose |
|-------|-------|-------|---------|
| clarifier | Sonnet | Questions | Socratic requirements gathering |
| explorer | Haiku | Research | Fast codebase exploration |
| architect | Opus | Design | Architecture and interface design |
| critic | Sonnet | Design | Devil's advocate, challenges designs |
| security-reviewer | Sonnet | Design + Review | Security vulnerability analysis |
| planner | Sonnet | Plan | Task decomposition into small units |
| builder | Sonnet | Implement | One-task-at-a-time implementation |
| reviewer | Opus | Implement | Fresh-context code review per task |
| pr-creator | Sonnet | PR | PR assembly with audit trail |

## How the Orchestration Works

```
 YOU (human)
  │
  │  /qrspi "add rate limiting to the payment endpoint"
  │
  ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: QUESTIONS                                         │
│  ┌────────────┐                                             │
│  │ Clarifier  │──▶ reports/01-requirements.md               │
│  │  (Sonnet)  │                                             │
│  └────────────┘                                             │
│  "Requirements ready. Approve, request changes, or reject?" │
└─────────────────────────┬───────────────────────────────────┘
                          │ ✅ Human approves
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 2: RESEARCH (parallel agent team)                    │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐              │
│  │ Explorer 1 │ │ Explorer 2 │ │ Explorer 3 │  (all Haiku) │
│  │  core code │ │   APIs &   │ │  tests &   │              │
│  │            │ │   deps     │ │   infra    │              │
│  └─────┬──────┘ └─────┬──────┘ └─────┬──────┘              │
│        └──────────┬───┘──────────────┘                      │
│                   ▼                                         │
│        Orchestrator synthesizes all 3 findings              │
│        ──▶ reports/02-exploration.md                        │
│  "Exploration complete. Approve to proceed to Design?"      │
└─────────────────────────┬───────────────────────────────────┘
                          │ ✅ Human approves
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 3: DESIGN (parallel debate team)                     │
│  ┌────────────┐    messages    ┌────────────┐               │
│  │ Architect  │◄──────────────▶│   Critic   │               │
│  │   (Opus)   │    challenges  │  (Sonnet)  │               │
│  │            │◄──────────────▶│            │               │
│  │            │    security    ┌────────────┐               │
│  │            │◄──────────────▶│  Security  │               │
│  └────────────┘                │  Reviewer  │               │
│        │                       │  (Sonnet)  │               │
│        ▼                       └────────────┘               │
│  Architect revises until team converges                     │
│  ──▶ reports/03-design.md                                   │
│  "Design finalized. Approve to proceed to Plan?"            │
└─────────────────────────┬───────────────────────────────────┘
                          │ ✅ Human approves
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 4: PLAN                                              │
│  ┌────────────┐                                             │
│  │  Planner   │──▶ tasks.json + reports/04-plan.md          │
│  │  (Sonnet)  │   (2-5 min tasks, ordered by dependency)   │
│  └────────────┘                                             │
│  "12 tasks planned. Approve, reorder, split, or remove?"    │
└─────────────────────────┬───────────────────────────────────┘
                          │ ✅ Human approves
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 5: IMPLEMENT (pipelined team, per task)              │
│                                                             │
│  For each task:                                             │
│  ┌────────────┐                                             │
│  │  Builder   │──▶ implements task N                        │
│  │  (Sonnet)  │──▶ runs tests                              │
│  └─────┬──────┘                                             │
│        │ "Here's what changed (diff). Approve commit?"      │
│        │  ✅ Human approves commit                          │
│        ▼                                                    │
│  ┌────────────┐                                             │
│  │  Reviewer  │──▶ reviews in fresh context                 │
│  │   (Opus)   │──▶ reports/review-task-N.md                 │
│  └─────┬──────┘                                             │
│        │ PASS ──▶ "Task N approved. Proceed to N+1?"        │
│        │ FAIL ──▶ Builder fixes, re-submits                 │
│        │  ✅ Human approves next task                       │
│        ▼                                                    │
│  (repeat for all tasks)                                     │
└─────────────────────────┬───────────────────────────────────┘
                          │ All tasks approved
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  Phase 6: PULL REQUEST                                      │
│  ┌────────────┐                                             │
│  │ PR Creator │──▶ assembles PR with full audit trail       │
│  │  (Sonnet)  │   (links to all reports/)                   │
│  └────────────┘                                             │
│  "PR ready. Review and merge?"                              │
└─────────────────────────────────────────────────────────────┘
```

Every arrow marked ✅ is a **human gate** — the pipeline stops and waits for your explicit approval. You are always in control.

## Key Features

- **Human gates everywhere** — every phase transition and every commit requires explicit human approval
- **Pre-commit review** — builder presents all changes with git diff before committing, waits for approval
- **Explicit next-action statements** — every agent declares what happens next before stopping (e.g., "When Explorer 3 completes, I will synthesize findings and wait for approval before Phase 3")
- **Session continuity** — `progress.md`, `tasks.json`, and `reports/` enable resume across sessions
- **Anti-loop guard** — agents stop reading after 10 files and produce output with what they have
- **Portable** — works as a Claude Code plugin (multi-agent) or as a standalone CLAUDE.md (single-agent, works in Augment/Cursor/Copilot too)

## Installation (Claude Code Plugin)

### 1. Add the marketplace

```bash
claude plugin marketplace add /path/to/qrspi-orchestrator
```

### 2. Install the plugin

```bash
claude plugin install qrspi-orchestrator
```

### 3. Enable agent teams

Add to your `~/.zshrc` (or `~/.bashrc`):

```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### 4. Restart Claude Code

## Usage (Claude Code)

```
/qrspi <feature description>     # Start the full pipeline
/qrspi-status                    # Show current pipeline status
/qrspi-resume                    # Resume from last checkpoint
```

## Usage (Augment / Cursor / Copilot)

Copy `CLAUDE.md` from this repo into your target project root. Then tell the agent:

> "Start QRSPI pipeline for [feature description]"

This gives you the same 6-phase pipeline as a single-agent sequential workflow. You lose parallel agent teams but keep the structure, human gates, and pre-commit approvals.

## Updating the Plugin

After editing agent files, commands, or skills:

1. Bump the version in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
2. Run:
   ```bash
   claude plugin marketplace update qrspi-orchestrator
   claude plugin update qrspi-orchestrator@qrspi-orchestrator
   ```
3. Restart Claude Code

## Working Files

The plugin creates these in your target project during a pipeline run:

| File | Purpose | Updated By |
|------|---------|------------|
| `progress.md` | Activity log for session continuity | Every agent, every phase |
| `tasks.json` | Structured task list with status | Planner, Builder, Reviewer |
| `reports/01-requirements.md` | Validated requirements | Clarifier |
| `reports/02-exploration.md` | Codebase analysis | Explorer team |
| `reports/03-design.md` | Architecture and design | Architect |
| `reports/04-plan.md` | Human-readable task breakdown | Planner |
| `reports/review-task-N.md` | Per-task code review | Reviewer |

## Project Structure

```
qrspi-orchestrator/
├── .claude-plugin/
│   ├── plugin.json              # Plugin manifest
│   └── marketplace.json         # Local marketplace config
├── agents/                      # Agent definitions (model, tools, prompts)
│   ├── clarifier.md
│   ├── explorer.md
│   ├── architect.md
│   ├── critic.md
│   ├── security-reviewer.md
│   ├── planner.md
│   ├── builder.md
│   ├── reviewer.md
│   └── pr-creator.md
├── commands/                    # Slash commands
│   ├── qrspi.md                 # Main pipeline orchestrator
│   ├── qrspi-status.md          # Status reporter
│   └── qrspi-resume.md          # Session resumption
├── hooks/
│   └── hooks.json               # Stop + SubagentStop hooks for continuity
├── skills/
│   └── qrspi-pipeline/
│       └── SKILL.md             # Pipeline knowledge base
├── CLAUDE.md                    # Portable single-agent version
└── README.md
```

## Requirements

- Claude Code v2.1.32+
- For agent teams: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
