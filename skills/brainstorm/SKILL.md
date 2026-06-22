---
name: brainstorm
slash_command: "/runway brainstorm [feature]"
model: opus
description: 4-stage adaptive Q&A with contradiction pushback that produces (or updates) SESSION.md.
trigger_conditions:
  - "User runs /runway brainstorm"
  - "init skill completes and hands off"
  - "User runs /runway brainstorm [feature] for a mid-project re-run"
inputs:
  - name: ingested_inputs
    type: markdown
    source: brainstorm/input/
    required: false
    description: Reference material captured at init.
  - name: existing_session
    type: SESSION.md
    source: brainstorm/SESSION.md
    required: false
    description: Present only for mid-project [feature] re-runs (diff mode).
  - name: user_answers
    type: conversation
    source: user
    required: true
    description: Answers to the adaptive Q&A across the four stages.
outputs:
  - name: session_spec
    type: SESSION.md
    destination: brainstorm/SESSION.md
    description: The 6-section spec, written progressively and locked at the review gate.
errors:
  - code: E_UNRESOLVED_TENSION
    condition: A contradiction is raised but the user has not resolved it.
    message: "This tension must be resolved before we move to the next stage."
    recovery: Resolve the tension (records to Clarifications) or accept it as a ⚠️ Flag.
examples:
  - title: From scratch
    scenario: User chooses to answer questions rather than paste a spec.
    command: "/runway brainstorm"
    result: SESSION.md filled one section per stage; finalized after the Stage-4 review gate.
  - title: Mid-project feature
    scenario: User adds a feature to an existing project.
    command: "/runway brainstorm payments"
    result: Diff mode — shows what changed, appends/updates the relevant SESSION.md sections.
---

# brainstorm

> **Stub — implemented in PHASE_03.** Frontmatter is authoritative for docs generation.

## Behavior

The most complex skill. Runs an adaptive Q&A engine across four stages, building SESSION.md
progressively:

1. **Both-path entry** — ask whether to paste an existing spec or answer from scratch; both paths
   converge on the same SESSION.md.
2. **4 stages** — Problem & Purpose → Technical Constraints → Scope → Architecture. Each stage opens
   with one anchor question; follow-ups depend on answer depth.
3. **Contradiction pushback** — name tensions explicitly, flag unrealistic expectations, and block
   stage progression until resolved. Resolutions are recorded in `## Clarifications`.
4. **Spec review gate** — after Stage 4, summarize the full spec, get user confirmation, then finalize
   and lock SESSION.md for Orchestration.
5. **Mid-project re-run** — with `[feature]`, read the existing SESSION.md and enter diff mode.
