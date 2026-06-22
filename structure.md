# Project Structure — Runway

## runway/

```
runway/
├── PLUGIN.md                   # [PHASE_01] plugin manifest: 8 skills, 12 commands (9 available + 3 planned)
├── LICENSE                     # [PHASE_00] MIT
├── CONTRIBUTING.md             # [PHASE_00] contribution + dogfooding policy
├── CHANGELOG.md                # build changelog (one entry per completed phase)
├── EXECUTE_LOG.md              # [PHASE_01] append-only execution log (format header only)
├── structure.md                # This file
├── brainstorm/
│   └── SESSION.md              # Full project spec — the source of truth (2026-06-22)
├── phases/
│   ├── PLAN.md                 # 18-phase build plan (PHASE_00, PHASE_01 complete)
│   └── DASHBOARD.md            # Live build status board
├── schemas/                    # [PHASE_00] 7 YAML schema contracts (field-granular write domains)
│   ├── SKILL.md.schema.yaml
│   ├── PLUGIN.md.schema.yaml
│   ├── PHASE.schema.yaml
│   ├── BLUEPRINT.schema.yaml
│   ├── CLAUDE.md.schema.yaml
│   ├── SESSION.md.schema.yaml
│   └── DASHBOARD.md.schema.yaml
├── skills/                     # [PHASE_01] 8 SKILL.md stubs (full enriched front-matter, placeholder bodies)
│   ├── init/SKILL.md           #   sonnet — implemented in PHASE_02
│   ├── brainstorm/SKILL.md     #   opus   — implemented in PHASE_03
│   ├── orchestrate/SKILL.md    #   sonnet — implemented in PHASE_04
│   ├── plan/SKILL.md           #   opus   — implemented in PHASE_05
│   ├── review/SKILL.md         #   sonnet — implemented in PHASE_06
│   ├── launch/SKILL.md         #   sonnet — implemented in PHASE_07
│   ├── recover/SKILL.md        #   sonnet — implemented in PHASE_09
│   └── mission-control/SKILL.md#   sonnet — implemented in PHASE_10a/10b
├── template/                   # [PHASE_01] tree copied into user projects on /runway init
│   ├── CLAUDE.md               #   5 sections incl. System Constants (all defaults)
│   ├── brainstorm/SESSION.md   #   empty 6-section template
│   ├── brainstorm/input/.gitkeep
│   ├── phases/PLAN.md          #   empty plan template
│   ├── phases/DASHBOARD.md     #   empty board template
│   ├── phases/PHASE_TEMPLATE.md
│   ├── phases/blueprints/.gitkeep
│   ├── docs/structure.md
│   └── docs/CHANGELOG.md
├── scripts/
│   └── generate-docs.js        # [PHASE_00] docs generator skeleton (SKILL.md front-matter → site/docs/)
├── tests/
│   └── FIXTURE.yaml            # [PHASE_00] full pre-baked brainstorm session for the E2E test
├── site/
│   └── docs/agents.md          # generated (DO NOT EDIT) — Agent Reference for the 8 skills
└── .github/
    ├── workflows/ci.yml        # [PHASE_00] CI: unit / fixture-e2e / plugin-structure / docs-rebuild
    └── ISSUE_TEMPLATE/{bug,feature}.md
```

### Notes
- This is the active development folder for the **manual v1 build of Runway**.
- `brainstorm/SESSION.md` is the spec source of truth; `phases/PLAN.md` is the build sequence.
- `schemas/` are contracts; write domains are **field-granular** ((file, field) pairs), so plan/review
  can co-own a blueprint and launch/Mission Control can co-own DASHBOARD.md on disjoint fields.
- `site/docs/agents.md` is **generated** — never edit by hand; run `node scripts/generate-docs.js`.

### Status
- ✅ PHASE_00 Foundation — complete
- ✅ PHASE_01 Plugin scaffold — complete
- ▶️ Next: PHASE_02 (init) + PHASE_03 (brainstorm) — parallel-eligible; PHASE_12a (docs skeleton) also unblocked
