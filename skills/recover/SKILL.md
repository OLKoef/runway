---
name: recover
slash_command: "/runway recover"
model: sonnet
description: Diagnose a failed phase (context vs complexity), re-prompt ×3, then escalate.
trigger_conditions:
  - "User runs /runway recover"
  - "Mission Control detects a heartbeat gap or a failed Claude Code session"
inputs:
  - name: failure_output
    type: error-log
    source: failed Claude Code session
    required: true
    description: Error output / stack trace from the failed session.
  - name: phase_file
    type: phase
    source: phases/PLAN.md or phases/PHASE_[id].md
    required: true
    description: Phase description + acceptance criteria.
  - name: memory_bus
    type: CLAUDE.md
    source: CLAUDE.md
    required: true
    description: Project memory and System Constants (max_revision_attempts).
  - name: current_structure
    type: structure.md
    source: docs/structure.md
    required: true
    description: Filesystem state for diagnosis.
outputs:
  - name: reprompt
    type: agent-prompt
    destination: Claude Code sub-agent (re-launch)
    description: Enriched (context path) or decomposed (complexity path) re-prompt.
  - name: blocked_flag
    type: phase-frontmatter
    destination: phases/ phase file (status:blocked) + DASHBOARD.md via Mission Control
    description: Written on escalation after 3 failed attempts.
errors:
  - code: E_RECOVERY_EXHAUSTED
    condition: All max_revision_attempts re-prompts failed.
    message: "Recovery exhausted; phase marked blocked."
    recovery: Surface to user with what was tried, what failed, and a recommended next step.
examples:
  - title: Context failure
    scenario: Claude Code failed because it lacked a needed fact.
    command: "/runway recover"
    result: Diagnosis = context; enriched re-prompt with the missing information; phase retried.
  - title: Complexity failure
    scenario: The phase was too large/ambiguous.
    command: "/runway recover"
    result: Diagnosis = complexity; phase decomposed into smaller sub-tasks; re-prompted.
---

# recover

> **Stub — implemented in PHASE_09.** Frontmatter is authoritative for docs generation.

## Behavior

The Recovery Agent runs a two-step diagnosis with up to 3 re-prompts:

1. **Collect** failure context — error output, the phase file, CLAUDE.md, structure.md.
2. **Diagnose** — classify as a *context* failure (Claude Code lacked information) or a *complexity*
   failure (phase too large/ambiguous).
3. **Re-prompt** — context path: enriched re-prompt naming what was missing. Complexity path:
   decompose into smaller sub-tasks and re-prompt with the simplified breakdown.
4. **3-attempt loop** — read `max_revision_attempts` (default 3). On exhaustion, mark the phase
   `blocked`, update DASHBOARD.md via Mission Control, and surface the situation to the user.
