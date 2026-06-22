<!-- Phase file template. Copy this block into phases/PLAN.md (or a standalone phases/PHASE_<id>.md).
     Conforms to schemas/PHASE.schema.yaml. -->

```yaml
id: PHASE_00            # PHASE_<n> (+ optional letter suffix for splits, e.g. PHASE_10a)
title: <short title>
status: pending         # pending | in_progress | complete | blocked | at_risk
depends_on: []          # phase ids that must be complete first (DAG, no cycles)
writes_to:              # exclusive write domain — files/dirs this phase may modify
  - <path>
started_at: null        # ISO date set when entering in_progress
completed_at: null      # ISO date set when reaching complete
assigned_to: claude_code
risk_level: low         # low | medium | high
```

### Description
<One paragraph: what this phase produces and why.>

### Acceptance criteria
- [ ] <criterion — tick as completed>
- [ ] <criterion>
