---
name: plan
slash_command: "/runway plan"
model: opus
description: Turn a phase into a file-level blueprint — [S] embedded content, [L] 5-field specs.
trigger_conditions:
  - "User runs /runway plan"
  - "execute pipeline reaches the plan step for the next phase"
inputs:
  - name: build_plan
    type: PLAN.md
    source: phases/PLAN.md
    required: true
    description: The phase to plan (next pending phase in dependency order).
  - name: current_structure
    type: structure.md
    source: docs/structure.md
    required: true
    description: Current filesystem state — plan from what was actually built, not from PLAN.md alone.
  - name: schemas
    type: yaml
    source: schemas/
    required: true
    description: "[S] content must conform to the relevant schema."
outputs:
  - name: blueprint
    type: blueprint
    destination: phases/blueprints/PHASE_[id]_blueprint.md
    field: blueprint body (all fields except reviewed_by)
    description: BLUEPRINT.schema.yaml-compliant; one file per phase.
errors:
  - code: E_PREV_PHASE_INCOMPLETE
    condition: Phase N-1 is not complete when planning Phase N.
    message: "Cannot plan this phase until its dependencies are complete."
    recovery: Complete prior phases, then re-run plan.
examples:
  - title: Plan one phase
    scenario: Phase N-1 is complete; structure.md reflects it.
    command: "/runway plan"
    result: phases/blueprints/PHASE_NN_blueprint.md written; [S] files carry content, [L] files carry specs.
---

# plan

> **Stub — implemented in PHASE_05.** Frontmatter is authoritative for docs generation.

## Behavior

The Planning Agent runs **per-phase, sequentially** (Opus — blueprint quality drives Claude Code's
execution quality):

1. Before planning Phase N, read structure.md (current filesystem state) and Phase N-1's completed
   acceptance criteria. Plan from what was actually built.
2. Classify each file the phase touches as **[S] structural** (deterministic — PLUGIN.md, SKILL.md
   stubs, CLAUDE.md, templates) or **[L] logic** (skill implementation / agent behavior).
3. Read `schemas/` first; [S] content must conform to its schema.
4. Write a BLUEPRINT.schema.yaml-compliant blueprint: `toc[]` and `files[]` in correspondence; [S]
   files embed full content; [L] files carry all five spec fields (purpose, inputs, behavior,
   constraints, interface).
