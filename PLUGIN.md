---
name: runway
version: 0.1.0
description: Turn a rough project idea into a structured, AI-driven Claude Code development session.
skills:
  - name: init
    path: skills/init/SKILL.md
    model: sonnet
    description: Entry point — ingest files, pick a model, write the project template, detect existing projects.
  - name: brainstorm
    path: skills/brainstorm/SKILL.md
    model: opus
    description: 4-stage adaptive Q&A with contradiction pushback that produces SESSION.md.
  - name: orchestrate
    path: skills/orchestrate/SKILL.md
    model: sonnet
    description: Turn SESSION.md into PLAN.md with a phase dependency graph.
  - name: plan
    path: skills/plan/SKILL.md
    model: opus
    description: Turn each phase into a file-level blueprint ([S] content / [L] spec).
  - name: review
    path: skills/review/SKILL.md
    model: sonnet
    description: Validate blueprints against the schemas; 3-attempt revision loop before escalation.
  - name: launch
    path: skills/launch/SKILL.md
    model: sonnet
    description: Assemble the context packet and spawn Claude Code via the Agent SDK.
  - name: recover
    path: skills/recover/SKILL.md
    model: sonnet
    description: Diagnose a failed phase (context vs complexity), re-prompt ×3, then escalate.
  - name: mission-control
    path: skills/mission-control/SKILL.md
    model: sonnet
    description: Status, docs, and completion modes — keeps DASHBOARD.md, CLAUDE.md, and docs current.
commands:
  - command: "/runway init"
    skill: init
    status: available
    description: Initialize a Runway project — ingest inputs, pick a model, write the template.
  - command: "/runway brainstorm [feature]"
    skill: brainstorm
    status: available
    description: Run the brainstorm session (or, with [feature], a mid-project diff re-run).
  - command: "/runway orchestrate"
    skill: orchestrate
    status: available
    description: Generate PLAN.md (phases + dependency graph) from SESSION.md.
  - command: "/runway plan"
    skill: plan
    status: available
    description: Generate the next phase's file-level blueprint.
  - command: "/runway review"
    skill: review
    status: available
    description: Validate the current blueprint against the schemas.
  - command: "/runway execute"
    skill: execute
    status: planned
    description: Primary pipeline — run plan → review → launch per phase with live progress. (skill lands in PHASE_08)
  - command: "/runway launch"
    skill: launch
    status: available
    description: Spawn a Claude Code sub-agent for a reviewed blueprint (power-user / debug).
  - command: "/runway recover"
    skill: recover
    status: available
    description: Diagnose and recover a failed phase.
  - command: "/runway status"
    skill: mission-control
    status: available
    description: Mission Control status mode — rebuild DASHBOARD.md, flag at_risk/blocked phases.
  - command: "/runway document"
    skill: mission-control
    status: available
    description: Mission Control docs mode — update README, structure.md, CHANGELOG.md, inline comments.
  - command: "/runway resume"
    skill: resume
    status: planned
    description: Resume from the last in_progress / next pending phase. (skill lands in PHASE_11)
  - command: "/runway upgrade"
    skill: upgrade
    status: planned
    description: Run a semver-driven migration (with --dry-run). (skill lands in PHASE_12c)
---

# Runway

Runway is a Cowork-native plugin that creates the onramp between *"I have an idea"* and
*"I have a running Claude Code session with good context."* It captures intent before any code is
written, builds a shared mental model through structured brainstorming, and produces the spec and
phase plan that ground every Claude Code session from the start.

Coordination between every agent happens through **file writes only** — no agent-to-agent messaging.

## Skills

| Skill | Model | Description |
|---|---|---|
| init | sonnet | Entry point — ingest files, pick a model, write the project template, detect existing projects. |
| brainstorm | opus | 4-stage adaptive Q&A with contradiction pushback that produces SESSION.md. |
| orchestrate | sonnet | Turn SESSION.md into PLAN.md with a phase dependency graph. |
| plan | opus | Turn each phase into a file-level blueprint ([S] content / [L] spec). |
| review | sonnet | Validate blueprints against the schemas; 3-attempt revision loop. |
| launch | sonnet | Assemble the context packet and spawn Claude Code via the Agent SDK. |
| recover | sonnet | Diagnose a failed phase (context vs complexity), re-prompt ×3, escalate. |
| mission-control | sonnet | Status, docs, and completion modes — keeps DASHBOARD.md, CLAUDE.md, and docs current. |

## Commands

| Command | Skill | Status | Description |
|---|---|---|---|
| `/runway init` | init | available | Initialize a Runway project. |
| `/runway brainstorm [feature]` | brainstorm | available | Run brainstorm (or a mid-project diff re-run). |
| `/runway orchestrate` | orchestrate | available | Generate PLAN.md from SESSION.md. |
| `/runway plan` | plan | available | Generate the next phase's blueprint. |
| `/runway review` | review | available | Validate the current blueprint. |
| `/runway execute` | execute | planned | Primary pipeline: plan → review → launch (PHASE_08). |
| `/runway launch` | launch | available | Spawn a Claude Code sub-agent (power-user / debug). |
| `/runway recover` | recover | available | Diagnose and recover a failed phase. |
| `/runway status` | mission-control | available | Mission Control status mode. |
| `/runway document` | mission-control | available | Mission Control docs mode. |
| `/runway resume` | resume | planned | Resume from the last/next phase (PHASE_11). |
| `/runway upgrade` | upgrade | planned | Semver-driven migration (PHASE_12c). |

> **Planned** commands describe the public surface that ships across later build phases. Their skills
> are scaffolded in the phase noted above; until then the command is documented but not yet wired.
