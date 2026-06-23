---
name: orchestrate
slash_command: "/runway orchestrate"
model: sonnet
description: Turn SESSION.md into PLAN.md — phases in dependency order with a valid DAG.
trigger_conditions:
  - "User runs /runway orchestrate"
  - "brainstorm finalizes SESSION.md"
  - "A mid-project /runway brainstorm [feature] adds scope that needs planning"
inputs:
  - name: session_spec
    type: SESSION.md
    source: brainstorm/SESSION.md
    required: true
    description: The finalized, review-approved 6-section spec. All six sections must be present.
  - name: schemas
    type: yaml
    source: schemas/
    required: true
    description: PHASE.schema.yaml (the contract every generated phase block must satisfy) and SESSION.md.schema.yaml (the 6-section input contract validated in step 1).
  - name: existing_plan
    type: PLAN.md
    source: phases/PLAN.md
    required: false
    description: Present on re-runs; drives the append-vs-new-file versioning decision.
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
    condition: SESSION.md is missing one or more of the six required sections.
    message: "SESSION.md is incomplete; run /runway brainstorm to finish it."
    recovery: Complete the brainstorm (all 6 sections + review gate), then re-run orchestrate.
  - code: E_EMPTY_SCOPE
    condition: SESSION.md Scope has no v1 (must-have) items.
    message: "No v1 scope to plan; add at least one must-have."
    recovery: Re-run /runway brainstorm to define v1 scope.
  - code: E_CYCLIC_GRAPH
    condition: The generated depends_on edges would form a cycle.
    message: "The phase graph has a cycle; dependencies were re-derived."
    recovery: Orchestrate breaks the cycle and re-validates; review the result.
examples:
  - title: First plan
    scenario: SESSION.md is complete with no unresolved flags.
    command: "/runway orchestrate"
    result: phases/PLAN.md written with an acyclic phase graph in dependency order; next step is /runway plan.
  - title: Unresolved flag
    scenario: SESSION.md still has a ⚠️ Flag.
    command: "/runway orchestrate"
    result: A dedicated resolution phase is generated before the affected implementation phases.
  - title: Mid-project versioning
    scenario: A small feature (≤3 phases) is added to an existing project.
    command: "/runway orchestrate"
    result: The new phases append to PLAN.md under a new section; a large feature would get phases/PLAN_[feature].md instead.
---

# orchestrate

The Orchestration Agent (Sonnet) converts the finalized `brainstorm/SESSION.md` into
`phases/PLAN.md`: an ordered set of build phases with a valid dependency graph. It is the bridge
between *what to build* (the spec) and *in what order* (the plan). It owns `phases/PLAN.md` and any
`phases/PLAN_[feature].md`; it never writes SESSION.md or DASHBOARD.md.

## Behavior

### 1. Read and validate SESSION.md
Read `brainstorm/SESSION.md`. Validate against `schemas/SESSION.md.schema.yaml`: all six sections
present (Problem & Purpose, Technical Constraints, Scope, Architecture, ⚠️ Flags, Clarifications). If
any is missing → `E_MISSING_SECTION`, stop, point the user to `/runway brainstorm`. If Scope has no
v1 must-have items → `E_EMPTY_SCOPE`.

### 2. Derive phases
From the Scope (v1 must-haves) and Architecture sections, decompose the work into phases. For each
phase, generate a block conforming to `schemas/PHASE.schema.yaml`:
- `id` (`PHASE_NN`, letter suffix for splits), `title`, `status: pending`.
- `depends_on` — the phase ids that must complete first.
- `writes_to` — the exclusive write domain (the files/dirs this phase touches). Two phases that may
  run in parallel must have **disjoint** `writes_to`.
- `started_at: null`, `completed_at: null`, `assigned_to: claude_code`, `risk_level` (low/medium/high).
Add a `### Description` paragraph and an `### Acceptance criteria` checklist to each.

### 3. Convert unresolved ⚠️ Flags into resolution phases
For every entry in SESSION.md's `## ⚠️ Flags` (anything other than the explicit "None" line), generate
a dedicated **resolution phase** and add its `id` to the `depends_on` of every implementation phase
that consumes the flagged decision — so the DAG (§4, §8), not file position, guarantees it runs first.
The resolution phase's own `depends_on` lists only its genuine prerequisites (often empty), so it
surfaces as runnable early. Orchestration does **not** block on flags — it schedules their resolution.

### 4. Build and validate the dependency graph
Assemble the `depends_on` edges into a directed graph and verify it is a **DAG**:
- Topologically sort; if a cycle is detected → `E_CYCLIC_GRAPH`, re-derive the offending edges
  (usually an over-specified mutual dependency) and re-validate.
- Every `depends_on` id must reference a phase that exists in the plan.
- Flag parallelizable groups: phases with no path between them **and** disjoint `writes_to`.

### 5. Write PLAN.md
Write `phases/PLAN.md` with: a header (`# <Project> — Build Phase Plan`), a `## Dependency Graph`
block (human-readable), `## Phases` (the per-phase YAML block + Description + Acceptance criteria), and
a `## Summary` table (phase | title | depends on | risk). Descriptions are human-readable; the YAML is
machine-checkable.

### 6. Plan approval (dual-mode)
Present the plan and support either approval path; **re-validate the graph after either**:
- **(a) Direct edit** — the user edits `phases/PLAN.md` by hand. Re-read it, re-validate sections +
  DAG + write-domain disjointness.
- **(b) Natural language** — the user describes changes ("merge phases 3 and 4", "add a phase for
  auth"). Apply them, then re-validate.
Loop until the user is satisfied; a valid DAG is a hard gate before handoff.

### 7. Plan versioning (re-runs / new features)
When `phases/PLAN.md` already exists (e.g. after a mid-project `/runway brainstorm [feature]`):
- **Small addition** — ≤3 new phases with **no new dependencies on existing phases** → append a new
  `## <Feature>` section to `phases/PLAN.md`.
- **Large feature** — otherwise → create `phases/PLAN_[feature].md` (same structure, scoped to the
  feature).
Either way, the new plan becomes discoverable to Mission Control, which lists it in the DASHBOARD.md
**Active Plans** index on its next status pass (orchestrate does not write DASHBOARD.md itself).

### 8. Handoff
On approval, the plan is ready for the Planning Agent — next step is `/runway plan` (or `/runway
execute` for the full pipeline). Phase execution starts at the first `pending` phase with no
unsatisfied `depends_on`.

## Constraints
- Exclusive owner of `phases/PLAN.md` and `phases/PLAN_[feature].md`. Never writes SESSION.md.
- Never writes DASHBOARD.md — Mission Control maintains the Active Plans index by scanning plan files.
- The dependency graph must be acyclic; every `depends_on` id must exist.
- Each phase block conforms to `schemas/PHASE.schema.yaml` (all required frontmatter fields).
- Does not block on unresolved flags — converts each into a resolution phase (§3).
- Neutral voice — clear, concise, no fluff.

## Examples
- **First plan:** a complete, flag-free SESSION.md → an acyclic PLAN.md with phases in dependency
  order and parallelizable groups noted; handoff to `/runway plan`.
- **Unresolved flag:** SESSION.md carries a `⚠️ Flag` about an undecided datastore → orchestrate emits
  a "Resolve datastore choice" phase before the phases that write to storage.
- **Mid-project versioning:** adding a 2-phase "CSV export" feature appends a `## CSV export` section
  to PLAN.md; a sprawling "multi-tenant" feature instead gets `phases/PLAN_multi-tenant.md`.
