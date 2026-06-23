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
  - name: plugin_template
    type: template-tree
    source: the plugin's template/ directory
    required: true
    description: The canonical scaffold copied into the user's project.
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
  - name: init_context
    type: CLAUDE.md-field
    destination: CLAUDE.md
    field: Current Status -> init note (mode + existing-project summary for brainstorm)
    description: One-time note telling brainstorm whether this is a greenfield or existing project.
errors:
  - code: E_TEMPLATE_CONFLICT
    condition: A template file already exists and differs from the template.
    message: "Existing file kept; template not overwritten."
    recovery: Review the existing file; merge manually if needed (init never overwrites).
  - code: E_INGEST_UNSUPPORTED
    condition: An attached file cannot be converted to markdown (unknown/binary format).
    message: "Couldn't convert <file>; copied a reference stub instead."
    recovery: Summarize the file by hand into brainstorm/input/, or proceed without it.
  - code: E_ALREADY_INITIALIZED
    condition: A complete Runway scaffold already exists (CLAUDE.md + phases/ + brainstorm/).
    message: "This project is already initialized."
    recovery: Run /runway brainstorm to continue, or /runway resume to pick up execution.
examples:
  - title: New project from a PDF brief
    scenario: User attaches a one-page brief and runs init in an empty directory.
    command: "/runway init"
    result: Brief converted to brainstorm/input/brief.md; template written; model recorded; brainstorm started.
  - title: Existing project
    scenario: Repo already has CLAUDE.md and structure.md.
    command: "/runway init"
    result: Existing-project mode; reads CLAUDE.md + README + structure.md; records context; tailors the first brainstorm question.
---

# init

`/runway init` is the entry point for every Runway project. It turns a directory (empty or existing)
into a Runway workspace: reference files become markdown, a model is chosen, the project scaffold is
written without clobbering anything, and the Brainstorm Agent is handed a primed context. It performs
the **one-time seed writes** to CLAUDE.md; after that, Mission Control owns CLAUDE.md.

## Behavior

Run the steps in order. Each step is idempotent and never overwrites user content.

### 1. Welcome
Print one short paragraph (neutral voice, no fluff): what Runway does, what is about to happen
(brainstorm → plan → execute), and what the output will be (a SESSION.md spec and a phase plan that
ground every Claude Code session). Keep it to ~3 sentences.

### 2. Detect project mode
**First, guard against re-initialization** (this check takes precedence over the branches below): if a
**complete** Runway scaffold already exists (`CLAUDE.md` *and* `phases/` *and* `brainstorm/`), stop
immediately with `E_ALREADY_INITIALIZED` and point the user to `/runway brainstorm` or `/runway
resume`. Do not scaffold or hand off.

Otherwise, check the project root for any of: `CLAUDE.md`, `structure.md` (or `docs/structure.md`),
`README*`.

- **Greenfield** (none found): proceed in new-project mode.
- **Existing project** (any found, but not a complete Runway scaffold): switch to **existing-project mode**:
  1. Read `CLAUDE.md` (if present), `README*`, and `structure.md`. If `structure.md` is absent,
     generate a directory listing (top 2–3 levels, skipping `.gitignore`d paths) as a stand-in.
  2. Distill a 3–6 line summary: what the project is, its stack, and its current shape.
  3. This summary becomes the `init_context` note (step 6) and tailors brainstorm's opening question
     (e.g. "I see an existing <stack> project — what are we adding or changing?" instead of the
     greenfield "What are we building?").

### 3. Model picker
Present the choice plainly:
- **Sonnet** — faster/cheaper; good default for most projects.
- **Opus** — deeper reasoning; better for large or ambiguous scopes.

Record the answer for step 6. If the user does not choose, default to **sonnet** and say so. (Note:
this sets the project's default execution model; the agent roster still uses Opus for brainstorm and
planning regardless — see CLAUDE.md Agent Roles.)

### 4. Write the template
Copy the plugin's `template/` tree into the project root. **Respect existing files**: for every target
path, if a file already exists, keep it untouched and record an `E_TEMPLATE_CONFLICT` note (do not
overwrite, do not merge automatically). Create only the missing files and directories. The tree
written: `CLAUDE.md`, `brainstorm/SESSION.md`, `brainstorm/input/`, `phases/PLAN.md`,
`phases/DASHBOARD.md`, `phases/PHASE_TEMPLATE.md`, `phases/blueprints/`, `docs/structure.md`,
`docs/CHANGELOG.md`. Writing the template first creates `brainstorm/input/` for step 5.

### 5. Ingest attached files
For each file the user attached, convert it to markdown and write it to
`brainstorm/input/<slugified-filename>.md`:
- **Text / Markdown** — copy as-is.
- **PDF / DOCX** — extract text to markdown (headings, lists, tables preserved where possible).
- **Images** — describe the image (and OCR any text) into a markdown stub that names the source file.
- **Unsupported / binary** — emit `E_INGEST_UNSUPPORTED`, write a reference stub naming the file, and
  continue. Never fail the whole init over one file.

These files are reference material for brainstorm; they are not the SESSION.md.

### 6. Seed CLAUDE.md (one-time writes)
Write only these two fields — init's exclusive seed domain. Mission Control owns the rest afterward,
including the Project Identity Name / one-liner (filled once brainstorm has produced SESSION.md):
- **Project Identity → Model**: the choice from step 3.
- **Current Status → init note**: the project mode and, for existing projects, the step-2 summary —
  so brainstorm opens with the right question.
Leave the Project Identity Name / one-liner at their template placeholders and System Constants at
their template defaults.

### 7. Hand off to brainstorm
Trigger the `brainstorm` skill (`/runway brainstorm`). Pass nothing through memory — brainstorm reads
`brainstorm/input/` and CLAUDE.md from disk (file-only coordination). Tell the user brainstorm is
starting.

## Constraints
- Never overwrite an existing file; co-existing user files always win.
- Only ever writes CLAUDE.md via the seed fields in step 6 — never DASHBOARD.md, never SESSION.md
  content (brainstorm owns SESSION.md).
- One file per attachment in `brainstorm/input/`; deterministic slug from the filename.
- If `template/` is unavailable, stop and report — init cannot scaffold without it.

## Examples
- **New project from a PDF brief:** user attaches `brief.pdf`, runs `/runway init` in an empty dir →
  `brainstorm/input/brief.md` created, scaffold written, model recorded, brainstorm starts.
- **Existing project:** repo has `CLAUDE.md` + `structure.md` → init reads them, records a summary in
  Current Status, writes only the missing scaffold files, and brainstorm opens with an
  "adding/changing" question instead of a greenfield one.
