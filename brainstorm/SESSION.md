# PROJECT: Runway
> Runway session — 2026-06-22 | Model: Opus

## Problem & Purpose
**One-liner:** A Cowork-native plugin that takes a rough project idea and turns it into a structured, AI-driven development session — from first prompt to Claude Code execution.

**Problem:** Solo devs and small teams start Claude Code sessions with vague prompts. The AI guesses at intent, makes structural decisions without context, and the session drifts. There's no structured onramp between "I have an idea" and "I have a running Claude Code session with good context." Runway creates that onramp: capturing intent before code is written, building a shared mental model, and producing a spec document that grounds every Claude Code session from the start.

**Target users:** Solo developers and small teams (2–5 people). Anyone who codes — not scoped to a specific stack, project type, or experience level.

**Success looks like:** A developer runs `/runway init`, completes a brainstorming session, approves a phase plan, and Claude Code begins executing with full context — without the developer having to re-explain the project at any point. Subsequent sessions pick up where the last left off.

## Technical Constraints
**Stack:** Cowork plugin (`.plugin` file). Skills written as SKILL.md files. Agents invoked via Cowork slash commands. Claude Agent SDK used by the Launcher Agent to spawn Claude Code as a sub-agent programmatically. Node.js for doc generation scripts (`/scripts/generate-docs.js`). MkDocs with Material theme for the docs site (GitHub Pages).

**Hard constraints:**
- Must work as a Cowork plugin — no separate CLI, no VS Code extension
- All agent coordination happens via file writes only — no direct agent-to-agent messaging
- Exclusive write domains enforced by design (not lockfiles) to prevent parallel session conflicts
- Claude Code never writes CLAUDE.md or DASHBOARD.md (Mission Control owns those)
- 3-attempt max on any revision loop (Blueprint Reviewer, Recovery Agent) — configurable as `max_revision_attempts` in CLAUDE.md

**Existing codebase:** Greenfield. `NewScaffoldingProject/brainstorm.md` serves as the source SESSION.md for v1 (dogfooding: v1 built manually, v2+ built with Runway itself).

## Scope

### v1 (must-have)
- `/runway init` — file ingestion (any format → markdown), model picker, folder template writer, existing project detection
- `/runway brainstorm` — 4-stage adaptive Q&A, contradiction pushback, spec review gate, both-path entry (paste or questions), mid-project re-run support
- `/runway orchestrate` — SESSION.md → PLAN.md with dependency graph; unresolved flags → generated phases
- `/runway plan` — PLAN.md → file-level blueprints per phase (structural [S] + logic [L] classification)
- `/runway review` — Blueprint Reviewer: [S] schema + [L] spec completeness validation, 3-attempt revision loop
- `/runway execute` — primary pipeline command (plan → review → launch in sequence), live progress stream, EXECUTE_LOG.md, pause/resume on failure
- `/runway launch` — Agent SDK sub-agent spawner (power user / debug)
- `/runway recover` — 2-step failure diagnosis (context vs complexity), re-prompt × 3, escalate to Mission Control
- `/runway status` — Mission Control status mode: DASHBOARD.md updates, parallel session management, at_risk/blocked flagging, hook heartbeat monitoring
- `/runway document` — Mission Control docs mode: README, structure.md, CHANGELOG, inline comments
- `/runway resume` — read DASHBOARD.md, find last in_progress/next pending phase, resume execution
- `/runway upgrade` — semver-driven migration, dry-run mode, step failure handler (revert/skip/retry)
- Schema system: 7 YAML schema files in `/schemas/` (SKILL.md, PLUGIN.md, PHASE, BLUEPRINT, CLAUDE.md, SESSION.md, DASHBOARD.md)
- CI: GitHub Actions — unit tests per skill + FIXTURE.yaml E2E test + plugin structure validation + docs rebuild check
- Docs site: MkDocs with Material, GitHub Pages, auto-generated from SKILL.md frontmatter

### v2+ (nice-to-have)
- Built with Runway itself (dogfooding claim earned only after v2 ships this way)
- Additional migration scripts as new versions ship
- Community-contributed question flows / skill extensions
- IDE integration (stretch)

### Out of scope
- CLI version
- VS Code extension
- Mobile interface
- Team collaboration features (multi-user sessions)
- Non-Anthropic model support

## Architecture
**Structure:** Cowork plugin with 8 SKILL.md files in `/skills/`, 7 schema YAML files in `/schemas/`, a project template in `/template/`, a Node.js docs generator in `/scripts/`, and a CI test fixture in `/tests/`. The plugin is distributed as a `.plugin` file installable in Cowork. User copies the `/template/` tree into their project on `/runway init`.

**Key patterns:**
- Multi-agent pipeline (8 agents + implicit Claude Code)
- File-based coordination — phase files with YAML frontmatter are the coordination bus
- Exclusive write domains — each agent owns specific files; overlap is impossible by design
- CLAUDE.md as shared memory bus — every agent reads it first; Mission Control keeps it current
- DASHBOARD.md as the live status board — Mission Control writes, Claude Code and user both read
- Mission Control hook heartbeat — Claude Code's existing tool-use hooks used as liveness signal; gap > `heartbeat_gap_minutes` (default: 10) triggers Recovery
- Blueprint dual-type system — structural [S] files get embedded content, logic [L] files get specs
- 3-attempt convention formalized in CLAUDE.md System Constants
- Schema override pattern — plugin bundles base schemas; project `/schemas/` directory can override (ESLint/Prettier pattern)
- Plan versioning — small additions append to PLAN.md; large features get their own `PLAN_[feature].md`
- CLAUDE.md compaction — Mission Control runs a compaction pass every 5 phases or when CLAUDE.md exceeds ~2000 words

**Integrations:**
- Claude Agent SDK — Launcher Agent spawns Claude Code sub-agent programmatically
- Claude Code hooks — post-write triggers Mission Control status mode; post-session triggers docs mode
- GitHub Actions — CI pipeline
- GitHub Pages — docs site hosting
- MkDocs with Material — docs site framework

**Agent roster:**

| Agent | Model | Role |
|---|---|---|
| Brainstorm Agent | Opus | 4-stage adaptive Q&A, contradiction pushback, spec review gate → SESSION.md |
| Orchestration Agent | Sonnet | SESSION.md → PLAN.md with dependency graph; handles unresolved flags |
| Planning Agent | Opus | PLAN.md → file-level blueprints per phase (sequential, reads structure.md before each) |
| Blueprint Reviewer | Sonnet | Validates blueprints: [S] schema + [L] 5-field spec. 3-attempt revision loop. |
| Launcher Agent | Sonnet | Assembles context packet → spawns Claude Code sub-agent via Agent SDK |
| Claude Code | Claude Code | Executes blueprints, ticks acceptance criteria, updates phase status, fires hooks |
| Recovery Agent | Sonnet | Diagnoses failures (context vs complexity), re-prompts × 3, escalates |
| Mission Control | Sonnet | Status + docs + completion modes; manages parallelism; CLAUDE.md compaction |

**Slash commands:** `/runway init`, `/runway brainstorm [feature]`, `/runway orchestrate`, `/runway plan`, `/runway review`, `/runway execute`, `/runway launch`, `/runway recover`, `/runway status`, `/runway document`, `/runway resume`, `/runway upgrade`

## ⚠️ Flags
_None — all tensions resolved during brainstorm._

## Clarifications
- **Docs agent vs overseer**: Merged into single Mission Control agent with three modes (status, docs, completion). One agent = one source of truth. No sync problems between separate agents writing the same files.
- **Parallel session cap**: Default 2 concurrent Claude Code sessions (not 3 — two is already noisy on a laptop). Configurable in CLAUDE.md. Mission Control checks API rate limits before spawning a second session.
- **Mission Control authority**: Intervention happens via file writes, not signals. Mission Control marks phases `blocked`; Claude Code reads that before starting. No complex agent-to-agent messaging required.
- **Blueprint Reviewer escalation**: After 3 failed revisions, surfaces both resolution paths to user: edit blueprint directly OR give natural language guidance to Planning Agent. Either path resets the attempt counter.
- **Planning Agent runs per-phase**: Sequential, not all-at-once. Before generating Phase N's blueprint, reads structure.md (current filesystem state) + Phase N-1's completed acceptance criteria. Plans from what was actually built, not from PLAN.md alone.
- **Unresolved SESSION.md flags**: Orchestration doesn't block — generates phases to explicitly resolve each flagged tension before the affected implementation phases.
- **Dogfooding**: v1 built manually (brainstorm.md as SESSION.md, handoff prompt as first blueprint). "Built with Runway since v1" claim added only after v2 ships this way — earned, not claimed.
- **SKILL.md schema is a prerequisite**: Enriched SKILL.md format (for docs auto-generation) must exist as `SKILL.md.schema.yaml` before Phase 1 writes any SKILL.md stubs. Resolved by making Phase 0 (Foundation) the explicit first phase.
- **Phase 10 split**: Mission Control was too large for one phase. Phase 10a = status mode; Phase 10b = docs + completion mode.
- **Phase 12 dependency gap**: Docs site skeleton (Phase 12a) can be built in parallel with skills (after Phase 1). Full docs generation (Phase 12b) requires all skills finalized (after Phase 11). Upgrade skill (Phase 12c) requires docs generator working (after Phase 12b).
