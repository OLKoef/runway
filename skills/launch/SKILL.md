---
name: launch
slash_command: "/runway launch"
model: sonnet
description: Assemble the context packet and spawn Claude Code as a sub-agent via the Agent SDK.
trigger_conditions:
  - "User runs /runway launch (power-user / debug)"
  - "execute pipeline reaches the launch step"
inputs:
  - name: reviewed_blueprint
    type: blueprint
    source: phases/blueprints/PHASE_[id]_blueprint.md
    required: true
    description: Must carry reviewed_by (pre-launch checklist).
  - name: memory_bus
    type: CLAUDE.md
    source: CLAUDE.md
    required: true
    description: Project memory; included in the context packet.
  - name: status_board
    type: DASHBOARD.md
    source: phases/DASHBOARD.md
    required: true
    description: Read to confirm no blocking flags; receives the new session ID.
outputs:
  - name: claude_code_session
    type: agent-session
    destination: Claude Code sub-agent (Agent SDK)
    description: Spawned with the context packet as initial prompt.
  - name: session_record
    type: DASHBOARD.md-field
    destination: phases/DASHBOARD.md
    field: Active Phase -> Session ID
    description: The only DASHBOARD.md cell the Launcher writes (heartbeat handoff); Mission Control owns the rest.
errors:
  - code: E_NOT_REVIEWED
    condition: Blueprint lacks reviewed_by.
    message: "Blueprint not reviewed; cannot launch."
    recovery: Run /runway review first.
  - code: E_DEP_INCOMPLETE
    condition: A depends_on phase is not complete, or DASHBOARD.md flags this phase blocked.
    message: "Dependencies not satisfied."
    recovery: Resolve the blocking phase, then re-launch.
examples:
  - title: Manual launch
    scenario: A reviewed blueprint exists and dependencies are complete.
    command: "/runway launch"
    result: Context packet assembled; Claude Code session spawned; session ID recorded on DASHBOARD.md.
---

# launch

> **Stub — implemented in PHASE_07.** Frontmatter is authoritative for docs generation.

## Behavior

The Launcher Agent assembles context and spawns Claude Code via the Claude Agent SDK:

1. **Pre-launch checklist** — blueprint has `reviewed_by`; all `depends_on` phases are `complete`; no
   DASHBOARD.md flag blocks this phase.
2. **Context packet** — approved blueprint + CLAUDE.md + DASHBOARD.md (not structure.md — too large;
   Claude Code reads it via tools if needed).
3. **Spawn** — programmatically start the Claude Code sub-agent with the packet as the initial prompt.
4. **Session tracking** — capture the session ID and record it on DASHBOARD.md so Mission Control can
   monitor the hook heartbeat.
