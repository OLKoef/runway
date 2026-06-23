# Changelog — Runway (build)

All notable changes during the manual v1 build of Runway. One entry per completed phase.
Format: phase id + title, the files it produced, and the date. Newest first.

## [PHASE_03] brainstorm skill — 2026-06-22
Implemented the Brainstorm Agent — the full SKILL.md body for `skills/brainstorm/`.

- 8-step engine: both-path entry (paste vs from-scratch, converging on one SESSION.md); 4 adaptive stages (Problem & Purpose → Technical Constraints → Scope → Architecture) with one anchor question each and capped follow-ups; continuous contradiction pushback that blocks stage progression and settles via resolve→Clarifications XOR accept→⚠️ Flags (never loops); progressive SESSION.md writer (SESSION.md.schema.yaml-conformant); Stage-4 review gate that locks the spec; mid-project diff-mode re-run.
- Adversarial-review fixes: corrected two cross-reference bugs (mid-project routed to the review gate instead of diff mode; paste path cited the writer instead of the gate); broadened the `init_context` input to declare the Project Identity → Model read that populates the SESSION.md header.

## [PHASE_02] init skill — 2026-06-22
Implemented the `/runway init` entry point — the full SKILL.md body for `skills/init/`.

- 7-step playbook: welcome → project-mode detection (greenfield vs existing, with a re-init guard) → model picker → non-overwriting template writer → file ingestion (PDF/DOCX/image/text → `brainstorm/input/`) → one-time CLAUDE.md seed → brainstorm handoff. Enriched frontmatter (3 inputs, 4 outputs, 3 errors).
- Adversarial-review fixes: made `E_ALREADY_INITIALIZED` an early precedence guard (was unreachable behind the "any found" branch); trimmed the CLAUDE.md seed to exactly its two declared fields (Model + Current Status init note), deferring Project Identity Name/one-liner to Mission Control; aligned the CLAUDE.md schema's seed-domain description to match.

## [PHASE_01] Plugin scaffold — 2026-06-22
Created the plugin definition, skill stubs, and the project template tree.

- Added `PLUGIN.md`: 8 skills + 12 commands (9 `available`, 3 `planned` — execute/resume/upgrade ship in later phases). Introduced a `status: available | planned` field on the PLUGIN schema so the manifest can advertise its full command surface before every skill is scaffolded; CI enforces the skill-bijection only for `available` commands.
- Added 8 `skills/*/SKILL.md` stubs (init, brainstorm, orchestrate, plan, review, launch, recover, mission-control) — full enriched front-matter, placeholder bodies, models per the SESSION.md roster.
- Added the full `template/` tree (CLAUDE.md with all System Constants, SESSION/PLAN/DASHBOARD/PHASE templates, docs templates, .gitkeeps) and `EXECUTE_LOG.md`.
- Adversarial verification (7 confirmed findings) drove follow-up fixes: write domains are now **field-granular** ((file, field) pairs) across the SKILL/DASHBOARD/CLAUDE schemas and skill stubs, so plan/review co-own a blueprint and launch/Mission Control co-own DASHBOARD.md on disjoint fields; `init`'s one-time CLAUDE.md seed write is now sanctioned; the CI docs-rebuild check switched from `git diff` to `git status --porcelain` (catches untracked generated files); the FIXTURE pushback turn moved to Stage 4 (where its tension is fully on the table).
- Verified: all schemas + PLUGIN.md + 8 SKILL.md parse as YAML and conform; PLUGIN ↔ skills bijection holds; generator regenerates `site/docs/agents.md` deterministically.

## [PHASE_00] Foundation — 2026-06-22
Prerequisite phase. Established the coordination contracts and project hygiene.

- Added 7 schema contracts under `schemas/`: SKILL.md, PLUGIN.md, PHASE, BLUEPRINT, CLAUDE.md, SESSION.md, DASHBOARD.md — all valid YAML, consistent meta-format.
- Added `scripts/generate-docs.js` (skeleton): zero-dependency, deterministic, discovers `skills/*/SKILL.md`, emits `site/docs/agents.md` (placeholder until skills exist).
- Added `tests/FIXTURE.yaml`: full pre-baked "Pocket CLI" brainstorm session (4 stages + 1 pushback turn) with structural `expected_outputs`.
- Added `.github/workflows/ci.yml`: four jobs — unit-tests (per skill), fixture-e2e, plugin-structure, docs-rebuild. Green at current state; extension points marked for PHASE_13.
- Added `LICENSE` (MIT), `CONTRIBUTING.md` (with dogfooding policy), and `.github/ISSUE_TEMPLATE/{bug,feature}.md`.
- Verified: all schemas + fixture + workflow parse as YAML; generator runs clean; generated docs committed.
