---
name: init
slash_command: "/runway init"
model: sonnet
description: Entry point — ingest attached files, pick a model, write the project template, detect existing projects.
trigger_conditions:
  - "User runs /runway init"
  - "First command in a fresh project directory"
inputs:
  - name: attached_files
    type: attached-files
    source: user (any format — PDF, DOCX, image, text)
    required: false
    description: Reference material; converted to markdown and written to brainstorm/input/.
  - name: existing_project_signals
    type: filesystem
    source: project root (CLAUDE.md, structure.md, README)
    required: false
    description: If present, init switches to existing-project mode.
outputs:
  - name: project_template
    type: template-tree
    destination: ./ (template/ tree copied into the project; existing files preserved)
    description: CLAUDE.md, brainstorm/, phases/, docs/ scaffolding.
  - name: ingested_inputs
    type: markdown
    destination: brainstorm/input/
    description: One markdown file per ingested attachment.
  - name: model_selection
    type: CLAUDE.md-field
    destination: CLAUDE.md
    field: Project Identity -> Model (one-time seed write at project creation)
    description: Selected model (sonnet|opus) recorded for downstream agents.
errors:
  - code: E_TEMPLATE_CONFLICT
    condition: A template file already exists and differs from the template.
    message: "Existing file kept; template not overwritten."
    recovery: Review the existing file; merge manually if needed.
examples:
  - title: New project from a PDF brief
    scenario: User attaches a one-page brief and runs init in an empty directory.
    command: "/runway init"
    result: Brief converted to brainstorm/input/brief.md; template written; brainstorm started.
  - title: Existing project
    scenario: Repo already has CLAUDE.md and structure.md.
    command: "/runway init"
    result: Existing-project mode; reads CLAUDE.md + README + structure.md; tailors the first brainstorm question.
---

# init

> **Stub — implemented in PHASE_02.** Frontmatter is authoritative for docs generation.

## Behavior

Entry point for every Runway project. On invocation, init:

1. Ingests any attached files, converting each to markdown under `brainstorm/input/`.
2. Presents a model picker (Sonnet vs Opus) and records the choice in CLAUDE.md.
3. Writes a short welcome message: what Runway does, what happens next, what the output will be.
4. Copies the `template/` tree into the project (never overwriting existing files).
5. Detects an existing project (CLAUDE.md or structure.md present) and switches to existing-project
   mode, reading current context and adjusting the brainstorm opening question.
6. Hands off to the brainstorm skill.
