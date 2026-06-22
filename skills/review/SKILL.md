---
name: review
slash_command: "/runway review"
model: sonnet
description: Validate a blueprint against the schemas; 3-attempt revision loop before escalation.
trigger_conditions:
  - "User runs /runway review"
  - "plan produces a new blueprint"
  - "execute pipeline reaches the review step"
inputs:
  - name: blueprint
    type: blueprint
    source: phases/blueprints/PHASE_[id]_blueprint.md
    required: true
    description: The blueprint to validate.
  - name: schemas
    type: yaml
    source: schemas/
    required: true
    description: BLUEPRINT, plus the per-file schema for each [S] file.
  - name: max_revision_attempts
    type: CLAUDE.md-constant
    source: CLAUDE.md (System Constants)
    required: true
    description: Revision-loop cap (default 3).
outputs:
  - name: reviewed_blueprint
    type: blueprint
    destination: phases/blueprints/PHASE_[id]_blueprint.md
    field: reviewed_by
    description: On approval, writes the reviewed_by field only; on rejection, returns actionable feedback (no write).
errors:
  - code: E_REVIEW_EXHAUSTED
    condition: max_revision_attempts failed revisions.
    message: "Blueprint still failing after N attempts."
    recovery: Escalate — user edits the blueprint directly OR gives natural-language guidance to plan. Either resets the counter.
examples:
  - title: Reject then approve
    scenario: A blueprint is missing a spec field on an [L] file.
    command: "/runway review"
    result: Specific feedback ("file X missing spec.constraints"); plan revises; re-validated; reviewed_by set.
---

# review

> **Stub — implemented in PHASE_06.** Frontmatter is authoritative for docs generation.

## Behavior

The Blueprint Reviewer is the quality gate between plan and launch:

1. Validate BLUEPRINT.schema.yaml compliance — required fields present, no unknown fields.
2. [S] content validation — non-empty, conforms to the relevant `schemas/` file.
3. [L] spec completeness — all five fields present and non-empty for every logic file.
4. ToC ↔ files cross-reference — bijective correspondence.
5. phase_id matches the phase under review.
6. On rejection, emit a specific, actionable feedback list (not just "invalid schema") and run the
   revision loop with plan, up to `max_revision_attempts`.
7. After 3 failures, escalate with both resolution paths. On approval, write
   `reviewed_by: blueprint-reviewer`.
