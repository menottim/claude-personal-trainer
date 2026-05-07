# Bootstrap CLAUDE.md — Setup Wizard

This is the bootstrap state of a freshly forked `claude-personal-trainer` repo. Once setup completes, this file is replaced with the user's personalized `CLAUDE.md`.

## On every conversation start

Check setup state in this order:

1. __If `data.json` exists at the repo root__: setup is complete but the bootstrap was somehow not replaced. Read the existing `data.json`, surface a warning that the bootstrap is still active, and suggest the user re-run setup or restore from `archive/` if available.
2. __Else if `.wizard-state.json` exists__: setup was paused. Read its contents, summarize what was captured, and ask the user if they want to resume or restart.
3. __Else__: setup has never been started. Greet the user, briefly explain what's about to happen (5-phase interview, ~20-35 minutes, generates a personalized repo), and ask if they're ready to begin.

## Wizard mode

When setup is in progress (cases 2 or 3 above), follow these files in order:

1. __`wizard/interview.md`__ — the 5-phase interview script. Conduct each phase as natural multi-turn conversation; do not read the prompts verbatim like a form.
2. After each phase, save state to `.wizard-state.json` so the user can pause and resume.
3. __`wizard/generation-rules.md`__ — once all 5 phases are complete, read this to compose the user's outputs (`data.json`, `knowledge/`, `index.html`, personalized `CLAUDE.md`).
4. __`wizard/claude-md-template.md`__ — the meta-template for the personalized `CLAUDE.md`. Populate from interview answers.
5. __`wizard/archive-rules.md`__ — relocate framework directories to `.claude-personal-trainer/` and self-overwrite this bootstrap file.

## Self-check before completing setup

Before declaring setup done:
- Validate `data.json` parses (`python3 -m json.tool data.json` should succeed)
- Verify `index.html` opens cleanly in a browser
- Verify the personalized `CLAUDE.md` is well-formed (no unfilled `{{PLACEHOLDER}}` tokens)
- Hand off to the user with: "Setup done. Tell me about your first activity."

## Hard rules during setup

- Do not log any "real" data during the wizard interview. The wizard is for configuration; data logging starts after setup completes.
- Do not write knowledge files until the user explicitly requests one or the personalized CLAUDE.md is in effect.
- Do not commit during the wizard except as the single "initial setup" commit at the very end.
- If the user wants to abort, save `.wizard-state.json` with a `status: "aborted"` flag and leave the repo otherwise untouched.
- Warn the user: `.wizard-state.json` is excluded by `.gitignore` (so a casual `git add .` won't catch it), but the file may contain personal disclosures (injuries, mental-health context, finances). Tell the user not to force-add it (`git add -f .wizard-state.json`) if the repo will be public.

## Git identity

If `git config user.name` is not set, ask the user for their name/email before the final commit. Do not assume an identity.
