---
name: mission-control
slash_command: "/runway status"
model: sonnet
description: Status, docs, and completion modes — keeps DASHBOARD.md, CLAUDE.md, and docs current; manages parallelism.
trigger_conditions:
  - "User runs /runway status (status mode)"
  - "User runs /runway document (docs mode)"
  - "Claude Code post-write hook fires (status mode)"
  - "A phase reaches complete (docs mode)"
  - "All phases reach complete (completion mode)"
inputs:
  - name: phase_files
    type: phase
    source: phases/ (PLAN.md + phase files)
    required: true
    description: Source of truth for the status board.
  - name: session_ids
    type: DASHBOARD.md-field
    source: phases/DASHBOARD.md
    required: false
    description: Live Claude Code session IDs for heartbeat monitoring.
  - name: system_constants
    type: CLAUDE.md-constant
    source: CLAUDE.md (System Constants)
    required: true
    description: heartbeat_gap_minutes, phase_timeout_minutes, parallel_session_cap, compaction_interval_phases.
outputs:
  - name: status_board
    type: DASHBOARD.md
    destination: phases/DASHBOARD.md
    field: all sections (full rewrite; excludes the Launcher-written Session ID cell)
    description: Atomic full rewrite — Active/Completed/Upcoming/Flags.
  - name: memory_bus
    type: CLAUDE.md
    destination: CLAUDE.md
    description: Kept current; compacted every compaction_interval_phases or when it exceeds ~2000 words.
  - name: project_docs
    type: markdown
    destination: README.md, docs/structure.md, docs/CHANGELOG.md
    description: Docs mode — README/structure/CHANGELOG plus inline comments on changed files.
errors:
  - code: E_HEARTBEAT_GAP
    condition: A session's hooks stop firing for > heartbeat_gap_minutes.
    message: "Session unresponsive; flagging at_risk and triggering Recovery."
    recovery: Recovery diagnoses; the phase is flagged on DASHBOARD.md.
examples:
  - title: Status rebuild
    scenario: A phase just changed status.
    command: "/runway status"
    result: DASHBOARD.md rewritten; any at_risk/blocked phases surfaced under Flags.
  - title: Completion
    scenario: All phases in PLAN.md are complete.
    command: "/runway status"
    result: "Completion mode: 'All phases done. What's next?' — offers /runway brainstorm [next feature]."
---

# mission-control

> **Stub — status mode in PHASE_10a, docs + completion modes in PHASE_10b.** Frontmatter is
> authoritative for docs generation.

## Behavior

One agent, three modes — the single source of truth for project state and docs.

**Status mode** (frequent; via Claude Code hooks)
- Hook heartbeat monitor: if hooks stop for > `heartbeat_gap_minutes`, flag `at_risk` and trigger Recovery.
- DASHBOARD.md updater: read all phase files, rebuild the board atomically (full rewrite).
- Parallel session scheduler: check `writes_to` conflicts across pending phases; approve conflict-free
  pairs up to `parallel_session_cap`; check rate limits before a second session.
- `at_risk` (timeout) and `blocked` (Recovery escalation) flagging.
- CLAUDE.md compaction every `compaction_interval_phases` completed phases or at ~2000 words.

**Docs mode** (after each phase reaches complete; via post-session hook)
- Update README, structure.md, CHANGELOG.md; add/update inline comments on changed files.

**Completion mode** (when all phases complete)
- Surface "All phases done. What's next?" and offer `/runway brainstorm [next feature]`.
