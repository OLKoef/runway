# Contributing to Runway

Thanks for your interest in Runway — the Cowork-native plugin that turns a rough project idea into a
structured, AI-driven development session.

## Ways to contribute

- **Bug reports** — use the [bug report template](.github/ISSUE_TEMPLATE/bug.md).
- **Feature requests** — use the [feature request template](.github/ISSUE_TEMPLATE/feature.md).
- **New skills** — propose additions to the skill/agent pipeline (a skill-proposal template lands in v0.1.0 polish).
- **Docs** — the hand-written pages live in `site/docs/`; the reference pages are generated (don't edit those by hand).

## Project layout

```
schemas/      YAML schema contracts (the validation layer between agents)
skills/       one SKILL.md per skill — the agent definitions
template/     the project tree copied into a user's repo on /runway init
scripts/      generate-docs.js — builds site/docs/ from SKILL.md front-matter
tests/        FIXTURE.yaml + (PHASE_13) the unit + E2E suites
site/docs/    docs site content (MkDocs Material); reference pages are GENERATED
.github/      CI workflow + issue templates
```

## Running the checks locally

Runway's checks are intentionally dependency-light.

```bash
# 1. Regenerate the docs and make sure nothing drifted
node scripts/generate-docs.js
git diff --exit-code site/docs        # must be clean

# 2. Validate every schema is well-formed YAML (needs Python + PyYAML)
python -m pip install --quiet pyyaml
python - <<'PY'
import glob, yaml
for p in sorted(glob.glob('schemas/*.yaml')):
    yaml.safe_load(open(p)); print('ok', p)
PY
```

The same checks run in CI (`.github/workflows/ci.yml`) across four jobs: unit tests per skill,
FIXTURE E2E, plugin-structure validation, and a docs-rebuild check. **Open a PR only when CI is green.**

## Conventions

- **Voice:** neutral — clear, consistent, no fluff. This applies to skill bodies, docs, and commit messages.
- **Schemas are contracts.** If you change a `schemas/*.yaml` file, update every artifact that conforms
  to it and rerun the docs generator.
- **Write domains are exclusive.** A skill writes only to the destinations declared in its `outputs`.
  Two skills must never write the same file.
- **Claude Code never writes `CLAUDE.md` or `DASHBOARD.md`** — Mission Control owns those.

## Dogfooding policy

Runway v1 was built **manually** (this repository), using its own `brainstorm/SESSION.md` as the spec and
`phases/PLAN.md` as the build plan. The claim **"built with Runway itself"** is *earned, not assumed*: it
applies only once a version ships by running Runway's own pipeline end-to-end (the v2 target). Until then,
contributions follow the manual flow described above.

## Code of conduct

Be respectful and constructive. Assume good faith. Keep discussion focused on the work.
