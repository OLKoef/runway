# DASHBOARD — Runway Build
> Live status board. Owned by Mission Control (during the manual v1 build, maintained by Claude Code). Atomic full rewrites only — never append.
> Last updated: 2026-06-22

## Active Plans
| Plan | Phases | Status |
|---|---|---|
| `phases/PLAN.md` | PHASE_00 → PHASE_14 (18 entries) | in_progress |

## Active Phase
| Phase | Title | Status | Session ID | Started |
|---|---|---|---|---|
| — | _none active_ | — | — | — |

## Completed
| Phase | Title | Completed |
|---|---|---|
| PHASE_00 | Foundation | 2026-06-22 |
| PHASE_01 | Plugin scaffold | 2026-06-22 |

## Upcoming
| Phase | Title | Depends on |
|---|---|---|
| PHASE_02 | init skill | PHASE_01 |
| PHASE_03 | brainstorm skill | PHASE_01 |
| PHASE_12a | docs site skeleton | PHASE_01 (parallel) |
| PHASE_04 | orchestrate skill | PHASE_02, PHASE_03 |

## ⚠️ Flags
_None._

## Notes
- This board tracks the **manual v1 build of Runway itself** (dogfooding target: v2 built with Runway).
- **Next:** PHASE_02 (init) + PHASE_03 (brainstorm) are parallel-eligible — disjoint write domains
  (`skills/init/` vs `skills/brainstorm/`), within `parallel_session_cap: 2`. PHASE_12a (docs skeleton)
  is also unblocked and independent.
