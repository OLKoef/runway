---
name: orchestrate
slash_command: "/runway orchestrate"
model: sonnet
description: Turn SESSION.md into PLAN.md — phases in dependency order with a valid DAG.
trigger_conditions:
  - "User runs /runway orchestrate"
  - "brainstorm finalizes SESSION.md"
inputs:
  - name: session_spec
    type: SESSION.md
    source: brainstorm/SESSION.md
    required: true
    description: The finalized 6-section spec. All six sections must be present.
outputs:
  - name: build_plan
    type: PLAN.md
    destination: phases/PLAN.md
    description: Phases with full YAML frontmatter per PHASE.schema.yaml + a dependency graph.
  - name: feature_plan
    type: PLAN_[feature].md
    destination: phases/PLAN_[feature].md
    description: Large features get their own plan file; small additions append to PLAN.md.
errors:
  - code: E_MISSING_SECTION
    condition: SESSION.md is missing one or more required sections.
    message: "SESSION.md is incomplete; run /runway brainstorm to finish it."
    recovery: Complete the brainstorm, then re-run orchestrate.
  - code: E_CYCLIC_GRAPH
    condition: Generated depends_on edges form a cycle.
    message: "The phase graph has a cycle; dependencies were re-derived."
    recovery: Orchestrate re-validates and breaks the cycle; review the result.
examples:
  - title: First plan
    scenario: SESSION.md is complete with no unresolved flags.
    command: "/runway orchestrate"
    result: phases/PLAN.md written with an acyclic phase graph in dependency order.
  - title: Unresolved flag
    scenario: SESSION.md still has a ⚠️ Flag.
    command: "/runway orchestrate"
    result: A dedicated resolution phase is generated before the affected implementation phases.
---

# orchestrate

> **Stub — implemented in PHASE_04.** Frontmatter is authoritative for docs generation.

## Behavior

1. Read and validate SESSION.md (all six sections present, or error).
2. Generate phases in dependency order; each phase carries full PHASE.schema.yaml frontmatter.
3. Build the dependency graph — `depends_on` edges form a valid DAG (no cycles).
4. For each unresolved `⚠️ Flag`, generate a dedicated resolution phase *before* the phases it affects
   (orchestrate does not block on flags).
5. Write PLAN.md (or PLAN_[feature].md for large features; small additions append). Keep the
   DASHBOARD.md active-plans index current.
6. Support dual-mode approval — the user edits PLAN.md directly, or describes changes in natural
   language; either way the graph is re-validated.
