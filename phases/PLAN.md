# Runway — Build Phase Plan
> Generated: 2026-06-22 | Total phases: 18 | Status: pending user handoff to Claude Code

---

## Dependency Graph

```
PHASE_00 → PHASE_01 → PHASE_02 ┐
                     PHASE_03 ┘ (parallel)
                     PHASE_12a  (parallel with 2–11)
           PHASE_02 → PHASE_04 (implicit: init must exist before orchestrate)
           PHASE_03 → PHASE_04
           PHASE_04 → PHASE_05
           PHASE_05 → PHASE_06
           PHASE_06 → PHASE_07
           PHASE_07 → PHASE_08
           PHASE_07 → PHASE_09
           PHASE_07 + PHASE_09 → PHASE_10a
           PHASE_10a → PHASE_10b
           PHASE_10a → PHASE_11
           PHASE_11 → PHASE_12b
           PHASE_12b → PHASE_12c
           all skills complete (incl. 12c) → PHASE_13
           PHASE_13 → PHASE_14
```

**Parallelizable groups (after PHASE_00 + PHASE_01 complete):**
- PHASE_02 + PHASE_03 run simultaneously (init and brainstorm are independent)
- PHASE_12a starts immediately after PHASE_01 (docs skeleton independent of skill logic)
- PHASE_05 → 06 → 07 → 08 → 09 must be sequential

---

## Phases

---

```yaml
id: PHASE_00
title: Foundation
status: complete
depends_on: []
writes_to:
  - schemas/
  - scripts/generate-docs.js
  - tests/FIXTURE.yaml
  - .github/workflows/ci.yml
  - LICENSE
  - CONTRIBUTING.md
  - .github/ISSUE_TEMPLATE/
started_at: 2026-06-22
completed_at: 2026-06-22
assigned_to: claude_code
risk_level: low
```

### Description
Prerequisite phase. Everything else depends on this. Produces all schemas (the coordination contract between Planning Agent and Blueprint Reviewer), the docs generator skeleton, the CI fixture, and GitHub hygiene files.

### Acceptance criteria
- [x] `/schemas/SKILL.md.schema.yaml` — enriched metadata: name, slash_command, model, trigger_conditions, inputs, outputs, errors, examples
- [x] `/schemas/PLUGIN.md.schema.yaml` — name, version, description, skills[], commands[]
- [x] `/schemas/PHASE.schema.yaml` — id, title, status, depends_on, writes_to, started_at, completed_at, assigned_to, risk_level
- [x] `/schemas/BLUEPRINT.schema.yaml` — phase_id, toc[], files[] with type: structural|logic; content for [S]; spec.{purpose,inputs,behavior,constraints,interface} for [L]
- [x] `/schemas/CLAUDE.md.schema.yaml` — 5 sections including System Constants with all configurable defaults
- [x] `/schemas/SESSION.md.schema.yaml` — 6 sections: problem, technical, scope, architecture, flags, clarifications
- [x] `/schemas/DASHBOARD.md.schema.yaml` — active phase, completed, upcoming, flags structure
- [x] `/scripts/generate-docs.js` skeleton — reads one SKILL.md frontmatter field, outputs a stub markdown page. Proves the generator architecture without requiring all skills.
- [x] `/tests/FIXTURE.yaml` — full pre-baked brainstorm session: seed_prompt, brainstorm_responses (all 4 stages), expected_outputs (structural validation, not content-exact)
- [x] `.github/workflows/ci.yml` — four jobs: unit tests per skill, FIXTURE E2E test, plugin structure validation, docs rebuild check
- [x] `LICENSE` — MIT
- [x] `CONTRIBUTING.md` — how to contribute, how to run tests, dogfooding policy
- [x] `.github/ISSUE_TEMPLATE/bug.md` — bug report template
- [x] `.github/ISSUE_TEMPLATE/feature.md` — feature request template

---

```yaml
id: PHASE_01
title: Plugin scaffold
status: complete
depends_on:
  - PHASE_00
writes_to:
  - PLUGIN.md
  - skills/
  - template/
started_at: 2026-06-22
completed_at: 2026-06-22
assigned_to: claude_code
risk_level: low
```

### Description
Creates the plugin definition, all 8 SKILL.md stubs conforming to the schema from Phase 0, and the full `/template/` directory tree that gets written to user projects on `/runway init`. Sets up the structure every subsequent phase builds into.

### Acceptance criteria
- [x] `PLUGIN.md` — plugin definition: name "runway", version 0.1.0, all 8 skills listed, all 12 slash commands listed
- [x] `/skills/init/SKILL.md` — stub conforming to SKILL.md.schema.yaml (all required frontmatter fields present; body can be placeholder)
- [x] `/skills/brainstorm/SKILL.md` — stub
- [x] `/skills/orchestrate/SKILL.md` — stub
- [x] `/skills/plan/SKILL.md` — stub
- [x] `/skills/review/SKILL.md` — stub
- [x] `/skills/launch/SKILL.md` — stub
- [x] `/skills/recover/SKILL.md` — stub
- [x] `/skills/mission-control/SKILL.md` — stub
- [x] `/template/CLAUDE.md` — 5 sections: Project Identity, Stack & Conventions, Agent Roles & Rules, System Constants (with all defaults: max_revision_attempts: 3, phase_timeout_minutes: 30, heartbeat_gap_minutes: 10, parallel_session_cap: 2, compaction_interval_phases: 5, git_integration: false), Current Status
- [x] `/template/brainstorm/SESSION.md` — empty 6-section template with field labels
- [x] `/template/brainstorm/input/.gitkeep`
- [x] `/template/phases/PLAN.md` — empty template with header and instructions
- [x] `/template/phases/DASHBOARD.md` — empty template with Active Phase / Completed / Upcoming / Flags structure
- [x] `/template/phases/PHASE_TEMPLATE.md` — phase file template with full YAML frontmatter + acceptance criteria section
- [x] `/template/phases/blueprints/.gitkeep`
- [x] `/template/docs/structure.md` — empty template
- [x] `/template/docs/CHANGELOG.md` — empty template with header
- [x] `EXECUTE_LOG.md` — empty file with format comment header

---

```yaml
id: PHASE_02
title: init skill
status: complete
depends_on:
  - PHASE_01
writes_to:
  - skills/init/SKILL.md
started_at: 2026-06-22
completed_at: 2026-06-22
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the `/runway init` skill. Entry point for all new projects. Handles file ingestion, model selection, project detection, and template writing. Parallel-eligible with Phase 03.

### Acceptance criteria
- [x] File ingestion pipeline — detects attached files (PDF, DOCX, image, text), converts each to markdown, writes to `brainstorm/input/[filename].md`
- [x] Model picker — presents Sonnet vs Opus choice; writes selection to CLAUDE.md Project Identity section
- [x] Welcome message — one short paragraph: what Runway does, what's about to happen, what the output will be
- [x] Template writer — copies `/template/` tree into user's project directory (respects existing files, doesn't overwrite)
- [x] Existing project detection — checks for CLAUDE.md or structure.md in project root; switches to "existing project mode" if found: reads CLAUDE.md + README + structure.md (or generates directory listing), adjusts Brainstorm Agent opening question
- [x] Triggers brainstorm skill on completion
- [x] SKILL.md fully conforming to enriched schema (all 8 frontmatter fields populated)

---

```yaml
id: PHASE_03
title: brainstorm skill
status: complete
depends_on:
  - PHASE_01
writes_to:
  - skills/brainstorm/SKILL.md
started_at: 2026-06-22
completed_at: 2026-06-22
assigned_to: claude_code
risk_level: high
```

### Description
Implements the Brainstorm Agent. The most complex skill — adaptive Q&A engine, pushback behavior, both-path entry, spec review gate, mid-project re-run support. Parallel-eligible with Phase 02.

### Acceptance criteria
- [x] 4-stage adaptive Q&A engine: Problem & Purpose → Technical Constraints → Scope → Architecture
- [x] Adaptive pacing — each stage opens with one anchor question; agent decides whether to follow up based on answer depth
- [x] Both-path entry — at start: "Paste existing spec or answer from scratch?" — both paths lead to the same SESSION.md output
- [x] Contradiction pushback — agent names tensions explicitly, flags unrealistic expectations, blocks stage progression until resolved
- [x] Resolved tensions written to SESSION.md `## Clarifications` section
- [x] SESSION.md progressive writer — adds one section per stage as conversation moves forward; document fills in progressively
- [x] Spec review gate — after Stage 4: agent summarizes full spec → user confirms → SESSION.md finalized and locked for Orchestration
- [x] Mid-project re-run support — `/runway brainstorm [feature]`: reads existing SESSION.md, enters diff mode, shows what changed, appends or updates relevant sections
- [x] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_04
title: orchestrate skill
status: pending
depends_on:
  - PHASE_02
  - PHASE_03
writes_to:
  - skills/orchestrate/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the Orchestration Agent. Reads SESSION.md and generates PLAN.md with a phase dependency graph. Handles unresolved flags and dual-mode plan approval.

### Acceptance criteria
- [ ] SESSION.md reader + parser — validates all 6 required sections present; errors if missing
- [ ] Phase plan generator — produces PLAN.md with phases in dependency order; each phase has full YAML frontmatter per PHASE.schema.yaml
- [ ] Dependency graph generator — `depends_on` fields form a valid DAG (no circular dependencies)
- [ ] Unresolved ⚠️ Flags handler — for each flag in SESSION.md, generates a dedicated "resolution phase" before the affected implementation phases
- [ ] PLAN.md writer — writes to `phases/PLAN.md` with human-readable phase descriptions + YAML frontmatter
- [ ] Dual-mode plan approval: (a) user edits PLAN.md directly, OR (b) user describes changes in natural language → Orchestration re-validates dependency graph after either path
- [ ] Plan versioning logic — small additions (≤3 phases, no new dependencies on existing phases) append to PLAN.md under new section; large features get `phases/PLAN_[feature].md`; DASHBOARD.md maintains index of all active plans
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_05
title: plan skill
status: pending
depends_on:
  - PHASE_04
writes_to:
  - skills/plan/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: high
```

### Description
Implements the Planning Agent. Generates file-level blueprints per phase. The quality of these blueprints directly determines Claude Code's execution quality — Opus is assigned here for a reason.

### Acceptance criteria
- [ ] Per-phase blueprint generator — produces one blueprint file per phase in `/phases/blueprints/PHASE_[id]_blueprint.md`
- [ ] Runs sequentially, not all-at-once — before generating Phase N blueprint, reads structure.md (current filesystem state) + Phase N-1 completed acceptance criteria
- [ ] Structural [S] vs logic [L] file classification — deterministic files (PLUGIN.md, SKILL.md stubs, CLAUDE.md, templates) get embedded content; skill implementations and agent behavior get specs
- [ ] Schema reader — reads `/schemas/` before generating any [S] content; [S] output must conform to its schema
- [ ] Blueprint writer — BLUEPRINT.schema.yaml compliant; all required fields present (phase_id, toc[], files[] with correct type + content/spec)
- [ ] [L] spec completeness — all 5 spec fields written: purpose, inputs, behavior, constraints, interface
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_06
title: review skill
status: pending
depends_on:
  - PHASE_05
writes_to:
  - skills/review/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the Blueprint Reviewer. Checklist-based quality gate between Planning Agent and Launcher. Validates before any Claude Code session starts.

### Acceptance criteria
- [ ] Blueprint schema validator — checks BLUEPRINT.schema.yaml compliance: all required fields present, no extra unknown fields
- [ ] [S] content validator — structural files have non-empty content; content conforms to the relevant schema from `/schemas/`
- [ ] [L] spec completeness checker — all 5 fields present and non-empty for every logic file: purpose, inputs, behavior, constraints, interface
- [ ] ToC cross-reference — every ToC entry has a matching files entry; every files entry has a matching ToC entry
- [ ] phase_id validator — blueprint phase_id matches the phase being reviewed
- [ ] Feedback generator — on rejection: specific, actionable list of what's wrong (not just "invalid schema")
- [ ] Revision loop — sends feedback to Planning Agent; Planning Agent revises; re-validates. Reads `max_revision_attempts` from CLAUDE.md (default: 3).
- [ ] Escalation — after 3 failed revisions: surfaces to user with both paths: (a) edit blueprint directly, (b) give natural language guidance to Planning Agent. Either path resets attempt counter.
- [ ] Writes `reviewed_by: blueprint-reviewer` to blueprint frontmatter on approval
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_07
title: launch skill
status: pending
depends_on:
  - PHASE_06
writes_to:
  - skills/launch/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the Launcher Agent. Assembles the context packet and spawns Claude Code as a sub-agent via the Claude Agent SDK.

### Acceptance criteria
- [ ] Context packet assembler — collects: approved blueprint + CLAUDE.md + DASHBOARD.md (not structure.md — too large; Claude Code reads via tools if needed)
- [ ] Agent SDK sub-agent spawner — programmatically spawns Claude Code sub-agent with context packet as initial prompt
- [ ] Session ID tracker — captures session ID from Agent SDK response; writes to DASHBOARD.md for Mission Control heartbeat monitoring
- [ ] Pre-launch checklist — verifies: blueprint has `reviewed_by` field set, all depends_on phases are `complete`, no phase flags in DASHBOARD.md blocking this phase
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_08
title: execute command
status: pending
depends_on:
  - PHASE_05
  - PHASE_06
  - PHASE_07
writes_to:
  - skills/execute/SKILL.md
  - EXECUTE_LOG.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements `/runway execute` — the primary user-facing command that orchestrates the full plan → review → launch pipeline. The command most users will run most of the time.

### Acceptance criteria
- [ ] Pipeline orchestrator — runs plan → review → launch in sequence for each phase; respects dependency order from PLAN.md
- [ ] Live progress streamer — outputs step status to Cowork chat in real time: `Planning Phase N... ✓`, `Reviewing... ✗ (2 issues)`, etc.
- [ ] EXECUTE_LOG.md writer — append-only, timestamped. Format: `[YYYY-MM-DD HH:MM] EXECUTE Phase NN — Step...   ✓/✗ result`
- [ ] Pause on review failure — stops pipeline, surfaces both resolution paths inline (edit blueprint / give guidance), waits for user input
- [ ] Resume after resolution — continues from the paused step after user resolves; resets attempt counter
- [ ] Parallel phase execution — after checking PLAN.md dependency graph + DASHBOARD.md conflict detection, spawns multiple Claude Code sessions for conflict-free parallel phases (up to `parallel_session_cap`)
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_09
title: recover skill
status: pending
depends_on:
  - PHASE_07
writes_to:
  - skills/recover/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the Recovery Agent. Two-step failure diagnosis with up to 3 re-prompt attempts before escalating.

### Acceptance criteria
- [ ] Failure context collector — gathers: error output/stack trace from failed Claude Code session, original phase file (description + acceptance criteria), CLAUDE.md, structure.md
- [ ] Diagnosis engine — classifies failure as: (a) context failure (Claude Code lacked information needed), or (b) complexity failure (phase was too large/ambiguous)
- [ ] Re-prompt generator (context path) — enriched re-prompt includes: phase file + CLAUDE.md + error explanation + explicit note on what was missing
- [ ] Re-prompt generator (complexity path) — decomposes phase into smaller sub-tasks; re-prompts with simplified breakdown
- [ ] 3-attempt loop — reads `max_revision_attempts` from CLAUDE.md; tracks attempt count; escalates if all fail
- [ ] Escalation — marks phase `blocked` in phase file frontmatter; updates DASHBOARD.md via Mission Control; surfaces to user with context: what was tried, what failed, recommended next step
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_10a
title: mission-control — status mode
status: pending
depends_on:
  - PHASE_07
  - PHASE_09
writes_to:
  - skills/mission-control/SKILL.md
  - template/phases/DASHBOARD.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: high
```

### Description
Implements Mission Control's status mode — the lightweight monitoring pass that runs frequently via Claude Code hooks. The liveness layer that keeps everything coherent.

### Acceptance criteria
- [ ] Hook heartbeat monitor — reads session IDs from DASHBOARD.md; if hooks stop firing for > `heartbeat_gap_minutes` (default: 10), flags phase `at_risk` and triggers Recovery
- [ ] DASHBOARD.md updater — reads all phase files in `/phases/`, builds live status board: Active Phase, Completed, Upcoming, Flags. Writes atomically (full rewrite, not append).
- [ ] Parallel session scheduler — on execute request: checks `writes_to` fields across all pending phases for conflicts; approves conflict-free pairs for simultaneous execution; enforces `parallel_session_cap` (default: 2); checks API rate limits before spawning second session
- [ ] at_risk flagging — phase in_progress beyond `phase_timeout_minutes` (default: 30) → mark `at_risk` in phase frontmatter + DASHBOARD.md Flags
- [ ] blocked flagging — phase `blocked` by Recovery escalation → DASHBOARD.md Flags + user-visible note
- [ ] CLAUDE.md compaction — every `compaction_interval_phases` (default: 5) completed phases OR when CLAUDE.md exceeds ~2000 words: summarizes completed phase history into `## History` section, removes stale detail
- [ ] SKILL.md fully conforming to enriched schema (shared with Phase 10b — same SKILL.md, extended)

---

```yaml
id: PHASE_10b
title: mission-control — docs + completion mode
status: pending
depends_on:
  - PHASE_10a
writes_to:
  - skills/mission-control/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Extends Mission Control with docs mode (triggered post-phase) and completion mode (triggered when all phases done). Adds to the same SKILL.md as Phase 10a.

### Acceptance criteria
- [ ] Docs mode trigger — fires after each phase reaches `complete` status (via post-session hook)
- [ ] README updater — keeps features, usage, and setup section current based on completed phases
- [ ] structure.md updater — hierarchical code map reflecting current filesystem state
- [ ] CHANGELOG.md updater — appends entry for completed phase: phase title, files changed, date
- [ ] Inline comment writer — adds or updates docstrings/inline comments in code files changed during the phase
- [ ] Completion mode — when all phases in PLAN.md reach `complete`: surfaces "All phases done. What's next?" prompt; offers `/runway brainstorm [next feature]` as entry point for next iteration
- [ ] SKILL.md updated with docs mode and completion mode sections (extends Phase 10a's SKILL.md, same file)

---

```yaml
id: PHASE_11
title: resume command
status: pending
depends_on:
  - PHASE_10a
writes_to:
  - skills/resume/SKILL.md
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: low
```

### Description
Implements `/runway resume`. First-class feature — pause and resume is documented prominently. Phase state persists in phase files so resumption is always possible.

### Acceptance criteria
- [ ] DASHBOARD.md reader — finds the last `in_progress` phase (session may have crashed) or next `pending` phase
- [ ] Session re-initialization — assembles context packet for the target phase (blueprint + CLAUDE.md + DASHBOARD.md), spawns new Claude Code session via Launcher
- [ ] Context handoff — includes EXECUTE_LOG.md tail in context packet so Claude Code knows what was already attempted
- [ ] EXECUTE_LOG.md continuation — appends `[timestamp] RESUME Phase NN — restarting after [gap]` before new session begins
- [ ] Handles missing in_progress — if no in_progress phase (clean pause), resumes from next pending phase in dependency order
- [ ] SKILL.md fully conforming to enriched schema

---

```yaml
id: PHASE_12a
title: docs site skeleton
status: pending
depends_on:
  - PHASE_01
writes_to:
  - mkdocs.yml
  - site/docs/
  - scripts/generate-docs.js
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: low
```

### Description
Builds the docs site skeleton and proves the generator architecture before all skills are finalized. Can start immediately after Phase 01 — runs in parallel with Phases 02–11.

### Acceptance criteria
- [ ] `mkdocs.yml` — MkDocs with Material theme config; GitHub Pages deployment; nav structure: Getting Started, Agent Reference, Command Reference, Schema Reference, Guides
- [ ] `/site/docs/index.md` — Getting Started narrative (hand-written: what Runway is, installation, first use)
- [ ] `/site/docs/agents.md` — placeholder: "Auto-generated from SKILL.md — run `node scripts/generate-docs.js`"
- [ ] `/site/docs/commands.md` — placeholder
- [ ] `/site/docs/schemas.md` — placeholder
- [ ] `/site/docs/guides/` — at least one hand-written guide (e.g., "Working with an existing codebase")
- [ ] `/scripts/generate-docs.js` — extended from Phase 00 skeleton: reads one full SKILL.md (init skill), generates a complete reference page for it. Proves the full generator pipeline works end-to-end.
- [ ] GitHub Pages deployment configuration in `.github/workflows/ci.yml` (add deploy step)

---

```yaml
id: PHASE_12b
title: docs site full generation
status: pending
depends_on:
  - PHASE_11
writes_to:
  - scripts/generate-docs.js
  - site/docs/
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Completes the docs generator — reads all 8 finalized SKILL.md files and all schemas to produce the full auto-generated reference. Requires all skills to be finalized (Phase 11 complete = all skills done).

### Acceptance criteria
- [ ] `/scripts/generate-docs.js` full implementation — reads all 8 SKILL.md files from `/skills/`, reads all 7 schema files from `/schemas/`
- [ ] Agent reference auto-generation — for each skill: name, model, trigger_conditions, inputs, outputs table
- [ ] Command reference auto-generation — for each skill: slash_command, description, errors, examples
- [ ] Schema reference auto-generation — for each schema YAML: field table with type and description
- [ ] Schema cross-references — inputs/outputs in agent reference link to their schema definitions
- [ ] CI docs rebuild check — CI job fails if any SKILL.md or schema file changes without a corresponding docs rebuild
- [ ] `/site/docs/CHANGELOG.md` — project changelog
- [ ] Sitemap and search index generated by MkDocs
- [ ] `/site/docs/` is generated output — `# DO NOT EDIT DIRECTLY` comment at top of each generated file

---

```yaml
id: PHASE_12c
title: upgrade skill
status: pending
depends_on:
  - PHASE_12b
writes_to:
  - skills/upgrade/SKILL.md
  - migrations/
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements `/runway upgrade`. Version migration system with dry-run, step failure handler, and semver-driven auto-trigger.

### Acceptance criteria
- [ ] `/skills/upgrade/SKILL.md` — fully conforming to enriched schema (adds to docs auto-generation)
- [ ] `/migrations/` directory — one migration script per version transition
- [ ] `/migrations/v0.1-to-v0.2.js` — first migration template: adds System Constants section to CLAUDE.md if missing; idempotent (safe to re-run)
- [ ] `runway_version` reader/writer — checks all Runway-owned project files for version field on any `/runway` command invocation
- [ ] Semver trigger logic — major version mismatch (X.y.z → X+1.y.z): block any command, show required upgrade prompt; minor/patch mismatch: log non-blocking warning to EXECUTE_LOG.md, continue
- [ ] `--dry-run` flag — plain-language summary of what would change: which files, which sections, why. No writes.
- [ ] Step failure handler — on migration step failure: pause; present 3 choices: (a) Revert all (undo all changes, stay on old version), (b) Skip and continue (mark step skipped, warn about inconsistency), (c) Retry (fix manually, re-attempt step). Each option clearly states its risk.

---

```yaml
id: PHASE_13
title: testing
status: pending
depends_on:
  - PHASE_12c
writes_to:
  - tests/
  - .github/workflows/ci.yml
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: medium
```

### Description
Implements the full CI test suite. Unit tests per skill + FIXTURE.yaml E2E implementation. All tests must pass before Phase 14.

### Acceptance criteria
- [ ] Unit test per skill — 8 tests, one per skill: known input → validated output structure (not content). Examples: Orchestration gets known SESSION.md → validates PLAN.md has correct dependency graph; Blueprint Reviewer gets known good + bad blueprints → validates correct accept/reject
- [ ] FIXTURE.yaml E2E test — runs full pipeline against `/tests/FIXTURE.yaml`: validates SESSION.md has all 6 sections, PLAN.md has valid dependency graph (no cycles), at least one blueprint generated, DASHBOARD.md created. Validation is structural, not content-exact.
- [ ] Plugin structure validation test — validates: all skills in PLUGIN.md have a corresponding SKILL.md file; all SKILL.md files pass schema validation; all schema YAML files are valid YAML
- [ ] Docs rebuild check test — validates: `node scripts/generate-docs.js` runs without error; output matches last committed `/site/docs/` (fails if SKILL.md changed without docs rebuild)
- [ ] All 4 CI jobs passing in `.github/workflows/ci.yml`
- [ ] Test run time under 5 minutes total for unit + structure + docs tests

---

```yaml
id: PHASE_14
title: GitHub repo polish
status: pending
depends_on:
  - PHASE_13
writes_to:
  - README.md
  - CONTRIBUTING.md
  - .github/ISSUE_TEMPLATE/
  - package.json
started_at: null
completed_at: null
assigned_to: claude_code
risk_level: low
```

### Description
Final phase. Polish the public-facing repo: minimal README pointing to docs site, contributing guide, issue templates, first release tag.

### Acceptance criteria
- [ ] `README.md` — minimal: project name, one-line description, install instructions (add plugin in Cowork), link to docs site. No feature list (that lives on docs site). Badge: CI status.
- [ ] `CONTRIBUTING.md` — how to contribute: fork, run tests (`node scripts/generate-docs.js` + CI), submit PR. Dogfooding policy: "New skills built with Runway itself since v2." Issue types: bug, feature, skill proposal.
- [ ] `.github/ISSUE_TEMPLATE/bug.md` — finalized (extends Phase 00 placeholder with real labels/assignees)
- [ ] `.github/ISSUE_TEMPLATE/feature.md` — finalized
- [ ] `.github/ISSUE_TEMPLATE/skill-proposal.md` — new: template for proposing new skill additions
- [ ] `package.json` — for scripts: `"docs": "node scripts/generate-docs.js"`, `"test": "..."` (CI-compatible test runner)
- [ ] Git tag `v0.1.0` created (or documented as the final manual step)
- [ ] All CI checks green on main branch

---

## Summary

| Phase | Title | Depends on | Risk |
|---|---|---|---|
| PHASE_00 | Foundation | — | low |
| PHASE_01 | Plugin scaffold | 00 | low |
| PHASE_02 | init skill | 01 | medium |
| PHASE_03 | brainstorm skill | 01 | high |
| PHASE_04 | orchestrate skill | 02, 03 | medium |
| PHASE_05 | plan skill | 04 | high |
| PHASE_06 | review skill | 05 | medium |
| PHASE_07 | launch skill | 06 | medium |
| PHASE_08 | execute command | 05, 06, 07 | medium |
| PHASE_09 | recover skill | 07 | medium |
| PHASE_10a | mission-control — status | 07, 09 | high |
| PHASE_10b | mission-control — docs + completion | 10a | medium |
| PHASE_11 | resume command | 10a | low |
| PHASE_12a | docs site skeleton | 01 (parallel) | low |
| PHASE_12b | docs site full generation | 11 | medium |
| PHASE_12c | upgrade skill | 12b | medium |
| PHASE_13 | testing | 12c | medium |
| PHASE_14 | GitHub repo polish | 13 | low |
