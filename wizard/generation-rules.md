# Wizard Generation Rules

Once `wizard/interview.md` is complete and `.wizard-state.json` contains all 5 phases, follow these rules to compose the user's personalized repo.

## Order of operations

1. Compose `data.json` from selected patterns
2. Pull seed knowledge modules from `knowledge-bank/` by tag
3. Compose `index.html` from widgets matching the selected patterns
4. Generate personalized `CLAUDE.md` from `wizard/claude-md-template.md`
5. Generate personalized `README.md`
6. Run self-check (validate JSON, verify HTML opens, no unfilled placeholders)
7. Apply `wizard/archive-rules.md` to relocate framework directories
8. Make a single git commit

## 1. Compose data.json

Read the user's selected patterns from `state.tracking_patterns` (Phase 3). For each:
- Read `patterns/<pattern-id>.json` for its schema fragment and example block
- Add a top-level array key in the user's `data.json` named conventionally:
  - `time-series-numeric` — user names it (e.g., `bodyLog`, `vocabSizeLog`, `moodLog`)
  - `event-log` — user names it (e.g., `activityLog`, `practiceLog`, `journalEntries`)
  - `streak-counter` — `streaks` (or user-named)
  - `periodic-review` — `reviews` (or user-named)
  - `tagged-collection` — user names it (e.g., `recipes`, `songIdeas`)
- Seed each array with one example entry derived from the user's stated context (not the pattern's generic example). e.g., for a fitness user: `{"date": "<today>", "weight": 220, "unit": "lbs", "notes": "Initial entry — adjust as needed"}`
- Add a top-level `meta` object containing: `domain`, `goals`, `user_profile` distillation, `created_date`, `tracker_version: "claude-personal-trainer-v1"`.

If patterns conflict (e.g., user picks two `time-series-numeric` for the same metric), ask a clarifying question rather than producing inconsistent JSON.

## 2. Pull seed knowledge modules

From the user's interview answers, derive a tag set:
- Phase 1 domain — primary tags (e.g., "strength" + "tendinopathy" + "in-season-athlete")
- Phase 1 constraints/injuries — additional tags
- Phase 1 success criteria — may add tags (e.g., "weight-loss" maps to "body-composition" tag)

For each tag in the derived set:
- Look it up in `knowledge-bank/_index.json`
- Copy matching `.md` files into the user's `knowledge/<topic>/` directory, preserving subdirectory structure
- Always include `general-evidence-discipline/how-to-cite.md` regardless of domain (it's the meta-module on citation discipline)

If no tags match (uncovered domain): create an empty `knowledge/` directory with a stub `README.md` that says: "This knowledge folder is empty. Claude will research and synthesize topics here as they come up in coaching, with verification per the evidence standards configured in CLAUDE.md."

## 3. Compose index.html

Skip this step entirely if `state.site_enabled` is false.

Otherwise:
- Read `state.site_shell` (Phase 5): pick `site-templates/dashboard-shell.html` or `site-templates/log-viewer-shell.html`
- For each pattern in `state.tracking_patterns`, read its `rendererHint` field (e.g., `progress-chart`)
- Inline the matching `widgets/<widget-id>.html` into the shell at the designated mount points
- Replace `{{DOMAIN_TITLE}}`, `{{USER_NAME}}`, etc. with values from interview state
- Save the composed result as `index.html` at the repo root

## 4. Generate personalized CLAUDE.md

Read `wizard/claude-md-template.md`. Populate every `{{PLACEHOLDER}}` token from the wizard state. Save the result as `CLAUDE.md` at the repo root, __overwriting the bootstrap__. Specifically:
- `{{DOMAIN}}` from state.domain
- `{{GOALS_PROSE}}` from state.goals (formatted as a paragraph)
- `{{USER_PROFILE}}` from state.user_profile (formatted)
- `{{COACHING_VOICE}}` from state.voice (rendered as guidance bullets)
- `{{FORBIDDEN_PATTERNS}}` from state.forbidden_patterns (as a list)
- `{{TRACKING_CONVENTIONS}}` derived from state.tracking_patterns (one section per pattern explaining how to log to it)
- `{{EVIDENCE_DISCIPLINE}}` from state.evidence_level (rendered as the matching ruleset block: see template for the three blocks)
- `{{WORKFLOW_RULES}}` derived from state (when to log, when to commit, when to write knowledge files, when to do periodic reviews)

## 5. Generate personalized README.md

Replace the bootstrap README with one personalized to the user's domain. Sections:
- Project (their domain + tagline)
- How to use (open in Claude Code, just talk to it)
- Data model (1-line description per pattern they're tracking)
- Knowledge (list of seed modules + invitation to grow)
- Re-running setup (delete `data.json` and `CLAUDE.md`, restore bootstrap from `.claude-personal-trainer/`)
- License (inherit MIT)

## 6. Self-check

Before completing:
- Run: `python3 -m json.tool data.json`: must succeed
- Open `index.html` in a headless verification (or just visually open in browser)
- Grep for `{{` in the personalized `CLAUDE.md` and `README.md`: must return no matches
- Grep for `{{` in any pulled `knowledge/` files: must return no matches (modules should not contain template placeholders)

If any check fails: report to user, do not commit, ask for direction.

## 7. Archive

Apply `wizard/archive-rules.md`.

## 8. Commit

Single commit using the user's git identity (ask if not configured). Suggested message:

```
Initial setup via claude-personal-trainer wizard

Domain: <state.domain>
Patterns: <state.tracking_patterns joined>
Site: <enabled|disabled>
Repo visibility recommended: <state.repo_privacy>
```
