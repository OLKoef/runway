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
| PHASE_02 | init skill | 2026-06-22 |
| PHASE_03 | brainstorm skill | 2026-06-22 |

## Upcoming
| Phase | Title | Depends on |
|---|---|---|
| PHASE_04 | orchestrate skill | PHASE_02, PHASE_03 (both complete → unblocked) |
| PHASE_12a | docs site skeleton | PHASE_01 (parallel) |
| PHASE_05 | plan skill | PHASE_04 |
| PHASE_06 | review skill | PHASE_05 |

## ⚠️ Flags
_None._

## Notes
- This board tracks the **manual v1 build of Runway itself** (dogfooding target: v2 built with Runway).
- **Next:** PHASE_04 (`orchestrate`) is now unblocked — its dependencies (PHASE_02 init + PHASE_03 brainstorm)
  are both complete. PHASE_12a (docs skeleton) remains independently available.
