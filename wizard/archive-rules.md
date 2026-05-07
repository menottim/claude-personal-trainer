# Wizard Archive Rules

After generation completes (per `wizard/generation-rules.md`), relocate framework directories so the user's repo top-level shows only their personalized files.

## Move these directories

Move (do not copy) the following from repo root to `.claude-personal-trainer/`:
- `wizard/`
- `patterns/`
- `widgets/`
- `site-templates/`
- `knowledge-bank/`
- `examples/`
- `docs/` (the upstream design spec + implementation plan; useful as reference but not user-facing)

After move:
- `.claude-personal-trainer/wizard/`
- `.claude-personal-trainer/patterns/`
- `.claude-personal-trainer/widgets/`
- `.claude-personal-trainer/site-templates/`
- `.claude-personal-trainer/knowledge-bank/`
- `.claude-personal-trainer/examples/`
- `.claude-personal-trainer/docs/`

## Save bootstrap files

Before overwriting `CLAUDE.md` (in step 4 of generation-rules), save a copy of the original bootstrap to:
- `.claude-personal-trainer/CLAUDE.md.bootstrap`

This allows the user to re-run setup later by restoring it.

Same for the original README:
- `.claude-personal-trainer/README.md.bootstrap`

## Delete or leave

- Delete `.wizard-state.json` after the final commit succeeds (it's no longer needed; setup is done)
- Leave `.gitignore`, `LICENSE`, `CONTRIBUTING.md` at the root (these are still useful to a forker)

## Verification

After the archive step, the repo root should contain ONLY:
- `README.md` (personalized)
- `LICENSE`
- `CONTRIBUTING.md`
- `CLAUDE.md` (personalized)
- `.gitignore`
- `data.json`
- `index.html` (if site enabled)
- `knowledge/` (user's pulled modules)
- `.claude-personal-trainer/` (framework + bootstraps)

Anything else at root indicates a generation bug. Report and don't commit.
