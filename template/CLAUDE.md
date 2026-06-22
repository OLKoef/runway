# CLAUDE.md

> Project memory bus. **Owned by Mission Control.** Every agent reads this first.
> Claude Code never writes this file. Conforms to schemas/CLAUDE.md.schema.yaml.

## Project Identity

- **Name:** <project name>
- **One-liner:** <one sentence on what this project is>
- **Target users:** <who it's for>
- **Model:** <sonnet | opus>   <!-- set by /runway init -->
- **runway_version:** 0.1.0

## Stack & Conventions

- **Languages / frameworks:** <fill in>
- **Conventions:** <naming, formatting, directory rules>
- **Layout:** <where code, docs, tests live>

## Agent Roles & Rules

Coordination happens through **file writes only** — no agent-to-agent messaging.

| Agent | Model | Owns (exclusive write domain) |
|---|---|---|
| brainstorm | opus | brainstorm/SESSION.md |
| orchestrate | sonnet | phases/PLAN.md |
| plan | opus | phases/blueprints/ |
| review | sonnet | blueprint `reviewed_by` |
| launch | sonnet | spawns Claude Code; DASHBOARD.md session id |
| Claude Code | — | implements blueprints; ticks acceptance criteria; **never** CLAUDE.md / DASHBOARD.md |
| recover | sonnet | phase `status: blocked` |
| mission-control | sonnet | CLAUDE.md, DASHBOARD.md, README, docs/ |

Rules:
- Each agent owns an exclusive write domain at **(file, field) granularity** — two agents touch the
  same file only on disjoint fields: plan writes the blueprint body, review only `reviewed_by`;
  Mission Control owns DASHBOARD.md, the Launcher writes only the Active-Phase Session ID cell.
- `/runway init` performs the one-time seed writes at project creation (this CLAUDE.md, including the
  Project Identity Model field); thereafter Mission Control owns CLAUDE.md.
- Claude Code never writes CLAUDE.md or DASHBOARD.md.
- Before starting a phase, check DASHBOARD.md for blocking flags.
- Revision loops (review, recover) stop after `max_revision_attempts`, then escalate.

## System Constants

| Constant | Default | Meaning |
|---|---|---|
| `max_revision_attempts` | 3 | Max attempts on any revision loop before escalation. |
| `phase_timeout_minutes` | 30 | A phase in_progress longer than this is flagged at_risk. |
| `heartbeat_gap_minutes` | 10 | Hook silence longer than this triggers Recovery. |
| `parallel_session_cap` | 2 | Max concurrent Claude Code sessions. |
| `compaction_interval_phases` | 5 | Compact CLAUDE.md every N completed phases (or at ~2000 words). |
| `git_integration` | false | Whether Runway commits/branches automatically. |

## Current Status

- **Active phase:** <none yet>
- **Status board:** phases/DASHBOARD.md
- **Where we are:** <short note; compacted history moves to ## History>
