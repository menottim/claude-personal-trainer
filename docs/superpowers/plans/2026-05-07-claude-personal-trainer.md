# `claude-personal-trainer` Implementation Plan

> __For agentic workers:__ REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

__Goal:__ Build a public, forkable Claude Code template repo that uses a bootstrap `CLAUDE.md` to interview each user and generate their personalized longitudinal-tracking-and-coaching repo from reusable patterns, widgets, and a topic-tagged knowledge bank.

__Architecture:__ Static-site + JSON-data + Claude-as-coach pattern (proven in `training-program/`), generalized: a `wizard/` directory of structured prompts that Claude reads to drive a 5-phase interview, plus reusable `patterns/` (JSON schemas), `widgets/` (HTML components), and `knowledge-bank/` (topic-tagged evidence modules) that the wizard composes into a personalized repo. After setup, framework directories relocate to `.claude-personal-trainer/`. v1 ships one bundled example (de-personalized strength-and-conditioning) demonstrating the end-state.

__Tech Stack:__ Markdown, JSON Schema, vanilla HTML/CSS/JS (no frameworks), Git. No build pipeline. Claude Code is the runtime.

__Security note:__ All widgets that render user-provided strings into the DOM use a small `esc()` HTML-escape helper rather than direct interpolation, to prevent XSS in the unlikely-but-possible scenario where `data.json` contains externally-sourced content (e.g., a user copy-pasting a knowledge-base entry from the web).

---

## File Structure (Day-0 ship)

| Path | Responsibility |
|------|---------------|
| `README.md` | Project overview, fork instructions |
| `LICENSE` | MIT |
| `CONTRIBUTING.md` | How to add patterns / widgets / knowledge / examples |
| `.gitignore` | Exclude `.wizard-state.json`, `.DS_Store`, etc. |
| `CLAUDE.md` | __Bootstrap.__ Detects no `data.json` to orchestrate wizard |
| `wizard/interview.md` | 5-phase interview script |
| `wizard/generation-rules.md` | Interview answers to composed outputs mapping |
| `wizard/claude-md-template.md` | Meta-template for personalized CLAUDE.md |
| `wizard/archive-rules.md` | Post-setup file relocation logic |
| `patterns/*.json` | 5 reusable JSON schema fragments |
| `widgets/*.html` | 5 reusable HTML+JS render components |
| `site-templates/*.html` | 2 minimal site shells |
| `knowledge-bank/_index.json` | Tag-to-files lookup |
| `knowledge-bank/<topic>/*.md` | ~10 seed evidence modules |
| `examples/strength-and-conditioning-recreational-athlete/` | End-state reference |

## File Structure (Day-N forker output)

| Path | Source |
|------|--------|
| `CLAUDE.md` | Generated from `wizard/claude-md-template.md` + interview answers |
| `data.json` | Composed from selected `patterns/*.json` |
| `knowledge/` | Subset of `knowledge-bank/` per interview tags + grow-over-time |
| `index.html` | Composed from `widgets/*.html` + a `site-templates/` shell |
| `README.md` | Personalized for their domain |
| `.claude-personal-trainer/` | All framework files relocated here |

---

# PHASE A: Skeleton + Wizard Prompts

Phase A produces a forkable repo where the bootstrap `CLAUDE.md` can run a 5-phase interview, even though generation logic (Phase D) hasn't been implemented yet. The skeleton is the scaffolding everything else hangs off.

## Task A.1: Initialize repo with root files

__Files:__
- Create: `~/claude-personal-trainer/README.md`
- Create: `~/claude-personal-trainer/LICENSE`
- Create: `~/claude-personal-trainer/.gitignore`
- Create: `~/claude-personal-trainer/CONTRIBUTING.md`

- [ ] __Step 1: Write minimal README.md__

```markdown
# claude-personal-trainer

A forkable Claude Code template that scaffolds a personalized longitudinal-tracking-and-coaching repo for any practice — fitness, language learning, financial planning, songwriting, woodworking, whatever.

## How it works

1. Fork this repo on GitHub
2. Clone locally
3. Open in [Claude Code](https://claude.com/claude-code)
4. Say anything to Claude. The bootstrap `CLAUDE.md` will detect no `data.json` exists and run an interactive setup wizard that interviews you across 5 phases:
   - Domain and Goals
   - Voice and Coaching Style
   - Tracking Preferences
   - Evidence Standards
   - Site / Artifact Preferences
5. Wizard generates your personalized `CLAUDE.md`, `data.json`, `knowledge/`, and `index.html`
6. You start using Claude as a coach in your domain. It logs to `data.json`, writes verified-citation knowledge files, and keeps a static dashboard in sync.

## See it working

Look at `examples/strength-and-conditioning-recreational-athlete/` for an end-state demo of what a fully-grown setup looks like.

## License

MIT
```

- [ ] __Step 2: Write LICENSE (MIT)__

```text
MIT License

Copyright (c) 2026 Menotti Minutillo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] __Step 3: Write .gitignore__

```text
.DS_Store
.wizard-state.json
node_modules/
*.log
```

- [ ] __Step 4: Write CONTRIBUTING.md (minimal stub, expanded in Task A.8)__

```markdown
# Contributing

This repo is a Claude Code template. Contributions welcome in four areas:

- __Patterns__ (`patterns/`): JSON schema fragments for `data.json` composition
- __Widgets__ (`widgets/`): HTML components that render specific patterns
- __Knowledge modules__ (`knowledge-bank/`): topic-tagged evidence syntheses
- __Examples__ (`examples/`): fully-grown reference repos for specific domains

See `CONTRIBUTING.md` Section 2+ (added in Task A.8) for detailed instructions.
```

- [ ] __Step 5: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add README.md LICENSE .gitignore CONTRIBUTING.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add root scaffolding: README, LICENSE, .gitignore, CONTRIBUTING stub"
```

---

## Task A.2: Write bootstrap `CLAUDE.md`

The bootstrap is the entry point. When a forker opens the repo in Claude Code and starts a conversation, this file tells Claude to detect setup state and route accordingly.

__Files:__
- Create: `~/claude-personal-trainer/CLAUDE.md`

- [ ] __Step 1: Write the bootstrap CLAUDE.md__

```markdown
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

## Git identity

If `git config user.name` is not set, ask the user for their name/email before the final commit. Do not assume an identity.
```

- [ ] __Step 2: Verify the file was written__

```bash
cd ~/claude-personal-trainer && wc -l CLAUDE.md && head -5 CLAUDE.md
```
Expected: ~50 lines, header `# Bootstrap CLAUDE.md — Setup Wizard`

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add CLAUDE.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add bootstrap CLAUDE.md with wizard orchestration logic"
```

---

## Task A.3: Write `wizard/interview.md` (5-phase interview script)

__Files:__
- Create: `~/claude-personal-trainer/wizard/interview.md`

- [ ] __Step 1: Create wizard directory and write interview.md__

```bash
mkdir -p ~/claude-personal-trainer/wizard
```

- [ ] __Step 2: Write the interview content__

```markdown
# Wizard Interview Script

Conduct each phase as multi-turn natural conversation. Do not list the prompts as a form. Follow up on interesting answers; skip prompts that don't apply. Save state to `.wizard-state.json` after each phase.

## Phase 1 — Domain and Goals (5-10 min)

Goal: understand what the user is trying to do, who they are, and what success looks like.

Open with: "Let's start with what you're trying to do. In a sentence or two, what's the practice or goal this repo is going to support?"

Then explore:
- What does success look like in 6 / 12 months?
- Who are you in this context: profession, time available per week, prior experience, constraints?
- Are there any injuries, conditions, limitations, or sensitivities that should shape advice?
- Who else is involved (coach, partner, group, just you)?

Branch on domain. If they say "fitness," dig into: sport, injury history, current strength baseline, training availability. If they say "language learning," dig into: target language, baseline level (A2? B1?), study time, immersion access. If novel, ask "what does a session of practice look like?" to ground the rest.

State to save: `domain`, `goals`, `user_profile`, `constraints`, `success_criteria`.

## Phase 2 — Voice and Coaching Style (3-5 min)

Goal: how should Claude talk to you?

Open with: "How do you want me to coach you? Direct and confrontational, or warmer? Should I push back when you're being lazy, or stay supportive?"

Then explore:
- Direct / warm / analytical / playful / dry: pick one or describe?
- AI-writing patterns to avoid? (em-dashes, "let's dive in," motivational fluff, excessive hedging, generic praise)
- Formatting: markdown bold style (`**bold**` vs. `__bold__`), prefer paragraphs or bullets, table-heavy or prose-heavy?
- Pronouns, name spellings, addressing style (first name? title?)
- Should I be encouraging on bad days, or honest about regression?

State to save: `voice`, `forbidden_patterns`, `formatting_prefs`, `name_handling`.

## Phase 3 — Tracking Preferences (5-10 min)

Goal: what data structure do they need? Map open-ended answers to patterns.

Open with: "What do you actually want me to log? Walk me through what a typical entry would contain."

Then explore:
- Frequency: daily, after each session, weekly retro, ad-hoc?
- Detail level: terse summaries or full transcripts?
- Numeric tracking (weight, mood, vocab count) maps to __time-series-numeric__ pattern
- Discrete events (workouts, sessions, songs) maps to __event-log__ pattern
- Habits / streaks maps to __streak-counter__ pattern
- Periodic retros maps to __periodic-review__ pattern
- Collections (recipes, ideas, cards) maps to __tagged-collection__ pattern
- Body / output stats: which matter, which don't?
- Confirm: list the patterns you'd recommend and ask for approval / additions / removals

State to save: `tracking_patterns` (array of pattern IDs to compose), `pattern_field_overrides` (any user customizations).

## Phase 4 — Evidence Standards (3-5 min)

Goal: how rigorous on citations? Set the discipline level for knowledge files.

Open with: "How rigorous do you want me to be on sources? Some users want strict peer-review citations with PMID/DOI. Others want me to share my best understanding without forcing a citation hunt."

Levels:
- __Strict__: every claim needs a verified peer-reviewed source. No citing without abstract verification. Mirrors the existing strength-and-conditioning repo's discipline.
- __Moderate__: cite when easy, label "Strong/Moderate/Emerging" evidence tier, speculate-with-flag when sources are weak.
- __Light__: Claude shares best understanding; flagged if uncertain but no citation requirement.

Also ask:
- Want a `knowledge/` folder that grows over time, or just live data?
- Web verification on knowledge writes (slower, higher fidelity), or fast/no-verification?
- Corrections-log discipline (record what was believed when, vs. silently revise)?

State to save: `evidence_level`, `web_verify_required`, `corrections_log_required`.

## Phase 5 — Site / Artifact Preferences (3-5 min)

Goal: do they want a visual dashboard? Public or private?

Open with: "Last thing — do you want a visual dashboard you can look at, or are data + Claude conversations enough?"

Then explore:
- Site or no site?
- Public via GitHub Pages, or local-only?
- __Privacy default__: strongly recommend private repo for any health, financial, or personal data. If data is sensitive and they say public, ask "are you sure?" once.
- Site shell preference: tabbed dashboard or single-page log viewer?
- Widget priorities (which patterns deserve hero placement?)

State to save: `site_enabled`, `site_visibility`, `site_shell`, `repo_privacy`.

## End of interview

After Phase 5, summarize all 5 phases concisely back to the user and ask: "Does this look right? I'm about to generate your repo from these answers." On confirmation, proceed to `wizard/generation-rules.md`.
```

- [ ] __Step 3: Verify__

```bash
cd ~/claude-personal-trainer && wc -l wizard/interview.md
```
Expected: ~70+ lines

- [ ] __Step 4: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/interview.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add wizard/interview.md with 5-phase script"
```

---

## Task A.4: Write `wizard/generation-rules.md`

__Files:__
- Create: `~/claude-personal-trainer/wizard/generation-rules.md`

- [ ] __Step 1: Write generation-rules.md__

```markdown
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
```

- [ ] __Step 2: Verify__

```bash
cd ~/claude-personal-trainer && wc -l wizard/generation-rules.md
```
Expected: ~80+ lines

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/generation-rules.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add wizard/generation-rules.md with composition logic"
```

---

## Task A.5: Write `wizard/claude-md-template.md` (meta-template)

__Files:__
- Create: `~/claude-personal-trainer/wizard/claude-md-template.md`

- [ ] __Step 1: Write the meta-template__

```markdown
# {{USER_DOMAIN_TITLE}} — LLM Instructions

## Overview

This is a personal {{DOMAIN}} tracker for {{USER_PROFILE_BRIEF}}. {{GOALS_PROSE}}

Live site (if applicable): {{SITE_URL}}

## Persona and Coaching Style

Act as the user's personal {{DOMAIN}} coach. {{COACHING_VOICE}}

### Voice constraints

{{FORBIDDEN_PATTERNS_SECTION}}

### Formatting

{{FORMATTING_PREFS}}

## Evidence Standards

{{EVIDENCE_DISCIPLINE}}

(Three possible blocks the wizard inserts here: strict / moderate / light. See `wizard/generation-rules.md` Section 4.)

### Strict block (peer-review-grade)

Every claim must follow this protocol:

1. __Check `knowledge/` first.__ Cite from verified sources listed there.
2. __If not in knowledge/__: search the web for trusted sources (PubMed, journal sites, systematic reviews, governing-body position stands). Confirm each paper exists and matches the claim by reading at least the abstract.
3. __If no support found:__ say so explicitly. "I can't find peer-reviewed support for this claim, so I'm not going to assert it." Offer mechanism-level reasoning if applicable, labeled as speculation.

Additional rules:
- __Cite what you recommend.__ Every change/adjustment needs a named source: author, year, journal, PMID/DOI.
- __Label evidence tier:__ Strong / Moderate / Emerging.
- __Corrections log:__ when new evidence supersedes prior claims, add a Corrections Log entry to the affected `knowledge/` file rather than silently revising.
- __No bro science / no hallucinated citations.__

### Moderate block

Cite when sources are easily available. Label evidence tier (Strong/Moderate/Emerging). For mechanism-plausible claims without strong sources, speculate-with-flag rather than assert. Corrections log encouraged but not required.

### Light block

Share best understanding without citation requirement. Flag uncertainty when it matters. Knowledge folder optional.

## Tracking Conventions

{{TRACKING_CONVENTIONS}}

(Per-pattern subsection. For each pattern in state.tracking_patterns, the wizard generates:
- The pattern's name and purpose
- The data.json key it maps to
- Required fields per entry
- Optional fields per entry
- Example entry
- When to log to it
- Activity-name format requirements if any)

## Workflow Rules

{{WORKFLOW_RULES}}

(Generated from interview state. Includes:
- When to log (immediately after a session, in a daily wrap-up, weekly batch?)
- When to commit (per session, daily, only on milestones?)
- When to write a new knowledge file (when a topic comes up >once, when user asks for it, never automatically?)
- When to do periodic reviews if periodic-review pattern is tracked
- Any domain-specific rules: e.g., for fitness, don't update `currentLifts` manually, derive from activityLog)

## Things NOT to Do

- Do NOT log data during conversation about __planning__; only log __actual__ entries
- Do NOT write knowledge files without user request unless workflow rules say otherwise
- Do NOT commit without permission unless workflow rules say otherwise
- {{ADDITIONAL_DOMAIN_DONTS}} (wizard adds domain-specific anti-patterns from interview)

## Re-running setup

To re-run the setup wizard, delete `CLAUDE.md` and `data.json`, then restore the bootstrap CLAUDE.md from `.claude-personal-trainer/CLAUDE.md.bootstrap` (kept by archive-rules). Optionally back up `data.json` first if you want to preserve any logs.

## Knowledge Files

The `knowledge/` folder contains evidence-synthesized modules. {{KNOWLEDGE_GROW_PROSE}} (wizard generates this based on grow-over-time setting and seed module count.)

---

## Schedule (if applicable)

{{DOMAIN_SCHEDULE_SECTION}}

(Wizard adds this section if the domain has a temporal structure: e.g., fitness phases, language-learning curriculum, financial-year boundaries. For domains without one, this section is omitted.)
```

- [ ] __Step 2: Verify__

```bash
cd ~/claude-personal-trainer && grep -c '{{' wizard/claude-md-template.md
```
Expected: > 10 placeholder occurrences (template has many tokens)

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/claude-md-template.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add wizard/claude-md-template.md meta-template"
```

---

## Task A.6: Write `wizard/archive-rules.md`

__Files:__
- Create: `~/claude-personal-trainer/wizard/archive-rules.md`

- [ ] __Step 1: Write archive-rules.md__

```markdown
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

After move:
- `.claude-personal-trainer/wizard/`
- `.claude-personal-trainer/patterns/`
- `.claude-personal-trainer/widgets/`
- `.claude-personal-trainer/site-templates/`
- `.claude-personal-trainer/knowledge-bank/`
- `.claude-personal-trainer/examples/`

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
```

- [ ] __Step 2: Verify__

```bash
cd ~/claude-personal-trainer && wc -l wizard/archive-rules.md
```
Expected: ~40 lines

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/archive-rules.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add wizard/archive-rules.md for post-setup file relocation"
```

---

## Task A.7: Expand README.md to full Day-0 version

__Files:__
- Modify: `~/claude-personal-trainer/README.md`

- [ ] __Step 1: Read current README__

```bash
cat ~/claude-personal-trainer/README.md
```

- [ ] __Step 2: Replace with full version__

```markdown
# claude-personal-trainer

A forkable [Claude Code](https://claude.com/claude-code) template that scaffolds a personalized longitudinal-tracking-and-coaching repo for any practice: fitness, language learning, financial planning, songwriting, woodworking, whatever.

## Why?

If you're using Claude as a coach for some practice you care about, you probably want:
- Your data captured longitudinally (not just in chat history)
- Claude maintaining a knowledge base of evidence-backed recommendations specific to your goals
- A persona tuned to your voice preferences
- A dashboard you can glance at

Building this from scratch every time is tedious. This template scaffolds it for you in ~30 minutes via a Claude-driven interview.

## How it works

1. __Fork this repo__ on GitHub
2. __Clone locally__: `git clone git@github.com:<you>/claude-personal-trainer.git`
3. __Open in Claude Code__: `cd claude-personal-trainer && claude`
4. __Say anything to Claude.__ The bootstrap `CLAUDE.md` will detect no `data.json` exists and run a 5-phase interactive setup:
   - __Phase 1: Domain and Goals__: what are you doing, what does success look like?
   - __Phase 2: Voice and Coaching Style__: how should Claude talk to you?
   - __Phase 3: Tracking Preferences__: what do you want to log?
   - __Phase 4: Evidence Standards__: citation discipline level
   - __Phase 5: Site / Artifact Preferences__: dashboard? public or private?
5. The wizard generates: a personalized `CLAUDE.md`, `data.json` shaped to your tracking preferences, seed `knowledge/` modules pulled from a topic-tagged bank, and (optionally) an `index.html` dashboard.
6. Day 2+: You talk to Claude as your coach. It logs to `data.json`, writes verified-citation knowledge files (with web search), and keeps the static dashboard in sync.

## See it working

`examples/strength-and-conditioning-recreational-athlete/` is a fully-grown reference repo (de-personalized) showing what a complete setup looks like for an in-season recreational athlete with achilles tendinopathy. Different domains will look different but share the same shape.

## Privacy

The wizard __strongly recommends__ keeping your resulting repo private if you're tracking health, financial, or personal data. Public is supported but explicit. GitHub Pages on private repos requires a paid GitHub plan; without it, your dashboard is local-only (which is fine for most users).

## Architecture

- __`CLAUDE.md` (bootstrap)__: drives the setup interview
- __`wizard/`__: interview script, generation rules, meta-templates
- __`patterns/`__: reusable JSON schema fragments composed into your `data.json`
- __`widgets/`__: reusable HTML components rendered in your `index.html`
- __`knowledge-bank/`__: topic-tagged seed evidence modules
- __`site-templates/`__: minimal generic site shells
- __`examples/`__: fully-grown reference repos

After setup, the framework directories relocate to `.claude-personal-trainer/` so your repo top-level is clean.

## Contributing

See `CONTRIBUTING.md` for how to add patterns, widgets, knowledge modules, or example domains.

## License

MIT (see LICENSE)
```

- [ ] __Step 3: Verify__

```bash
cd ~/claude-personal-trainer && wc -l README.md && head -1 README.md
```
Expected: ~50+ lines, header `# claude-personal-trainer`

- [ ] __Step 4: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add README.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Expand README with how-it-works, architecture, privacy"
```

---

## Task A.8: Expand `CONTRIBUTING.md` to full version

__Files:__
- Modify: `~/claude-personal-trainer/CONTRIBUTING.md`

- [ ] __Step 1: Replace with full version__

```markdown
# Contributing to claude-personal-trainer

This repo is a Claude Code template. Contributions welcome in four areas.

## 1. Adding a knowledge module

Knowledge modules are topic-tagged markdown evidence syntheses. Each module covers one well-bounded topic (e.g., "achilles HSR protocol", "spaced repetition", "mortgage amortization basics") with verified peer-reviewed citations.

__File location:__ `knowledge-bank/<topic-area>/<module-name>.md`

__Module structure:__

```
# <Module Title>

<1-paragraph problem statement>

## Key findings

- <claim>. Source: <author year, journal, PMID/DOI>. Evidence tier: <Strong|Moderate|Emerging>.

## Practical guidance

<actionable summary of what to do based on the findings>

## Corrections log

<empty initially; appended when claims are revised>

## References

<full citation list with PMIDs/DOIs>
```

__After adding the module:__ update `knowledge-bank/_index.json` with the new file path and its tags.

## 2. Adding a pattern

Patterns are JSON schema fragments that the wizard composes into a user's `data.json`.

__File location:__ `patterns/<pattern-id>.json`

__Pattern structure:__

```json
{
  "id": "<kebab-case-id>",
  "description": "<one-sentence description>",
  "schema": { "...JSON Schema fragment..." },
  "exampleBlock": [ "...one or more example entries..." ],
  "rendererHint": "<widget-id>"
}
```

The `rendererHint` points to a widget in `widgets/` that knows how to render this pattern.

## 3. Adding a widget

Widgets are vanilla HTML+CSS+JS components that render a specific pattern from `data.json`.

__File location:__ `widgets/<widget-id>.html`

__Constraints:__
- No framework dependencies (no React, Vue, etc.)
- Self-contained: HTML + inline CSS + inline JS, all in one file
- Reads `data.json` via `fetch('./data.json')` at the standard mount point
- Always uses an `esc()` helper for HTML-escaping any user-provided strings before they touch the DOM via innerHTML — XSS prevention is a hard requirement
- Under 200 LOC

__Widget skeleton (XSS-safe pattern):__

```html
<!-- WIDGET: <widget-id> -->
<!-- Renders: <pattern-id> -->
<style>
  /* widget-scoped CSS */
</style>

<h3>{{TITLE}}</h3>
<div id="<widget-id>-mount">Loading...</div>

<script>
(async () => {
  const esc = s => String(s == null ? '' : s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const data = await fetch('./data.json').then(r => r.json());
  // ... render logic, using `esc()` on every interpolated user value ...
})();
</script>
```

## 4. Adding an example

Examples are fully-grown reference repos demonstrating what a complete setup looks like in a specific domain.

__File location:__ `examples/<descriptive-domain-slug>/`

__Example structure:__

```
examples/<slug>/
  README.md            # what this example demonstrates
  CLAUDE.md            # personalized for the example domain (no real PII)
  data.example.json    # schema only, fake/synthetic data
  knowledge/           # synthesized evidence modules for this domain
  index.html           # working dashboard
```

__Critical:__ examples must contain __no real personal data__. Use synthetic / placeholder values throughout. PII review is required before any example PR is merged.

## Pull request process

1. Fork the repo
2. Create a feature branch (`feature/add-<thing>`)
3. Commit your changes following the style above
4. Open a PR; describe what you're adding and why
5. PRs that add knowledge modules MUST cite verified peer-reviewed sources (no fabricated references)
6. PRs that add examples MUST pass PII review

## License

By contributing, you agree your contributions are licensed under MIT.
```

- [ ] __Step 2: Verify__

```bash
cd ~/claude-personal-trainer && wc -l CONTRIBUTING.md
```
Expected: ~80+ lines

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add CONTRIBUTING.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Expand CONTRIBUTING.md with full guidance for all 4 contribution areas"
```

---

# PHASE B: Patterns + Widgets

All widgets in this phase use a small `esc()` HTML-escape helper to prevent XSS when rendering user-controlled `data.json` fields. The pattern is consistent across widgets.

## Task B.1: Create `patterns/time-series-numeric.json`

__Files:__
- Create: `~/claude-personal-trainer/patterns/time-series-numeric.json`

- [ ] __Step 1: Create patterns directory__

```bash
mkdir -p ~/claude-personal-trainer/patterns
```

- [ ] __Step 2: Write the pattern__

```json
{
  "id": "time-series-numeric",
  "description": "Track a numeric value over time with timestamps and optional notes. Use for: weight, mood scores, sleep hours, vocab counts, dollars-saved, anything quantifiable that you'll plot over time.",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["date", "value"],
      "properties": {
        "date": {"type": "string", "format": "date", "description": "ISO date YYYY-MM-DD"},
        "value": {"type": "number"},
        "unit": {"type": "string", "description": "Human-readable unit"},
        "notes": {"type": "string"}
      },
      "additionalProperties": true
    }
  },
  "exampleBlock": [
    {"date": "2026-05-07", "value": 220.5, "unit": "lbs", "notes": "AM weigh-in"},
    {"date": "2026-05-08", "value": 220.0, "unit": "lbs"}
  ],
  "rendererHint": "progress-chart",
  "naming": {
    "guidance": "Pick a name that describes what's being tracked, like 'bodyLog', 'moodLog', 'vocabSizeLog', 'savingsLog', 'sleepLog'. Use camelCase. The wizard will ask you for this.",
    "examples": ["bodyLog", "moodLog", "savingsLog"]
  }
}
```

- [ ] __Step 3: Verify it parses__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool patterns/time-series-numeric.json > /dev/null && echo OK
```
Expected: `OK`

- [ ] __Step 4: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add patterns/time-series-numeric.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add patterns/time-series-numeric.json"
```

---

## Task B.2: Create `patterns/event-log.json`

__Files:__
- Create: `~/claude-personal-trainer/patterns/event-log.json`

- [ ] __Step 1: Write the pattern__

```json
{
  "id": "event-log",
  "description": "Track discrete events with type/intensity/duration metadata. Use for: workouts, language practice sessions, songs written, journal entries, study sessions: anything that happens at a point in time and has structure.",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["date", "type"],
      "properties": {
        "date": {"type": "string", "format": "date"},
        "type": {"type": "string", "description": "Event category, domain-specific"},
        "activity": {"type": "string"},
        "duration": {"type": "string"},
        "intensity": {"type": "string", "enum": ["low", "moderate", "high", "max"]},
        "details": {"type": "object"},
        "notes": {"type": "string"}
      },
      "additionalProperties": true
    }
  },
  "exampleBlock": [
    {"date": "2026-05-07", "type": "workout", "activity": "Lower Body", "duration": "60 min", "intensity": "high"}
  ],
  "rendererHint": "timeline-table",
  "naming": {
    "guidance": "Pick a name that describes what's being logged. Use camelCase.",
    "examples": ["activityLog", "practiceLog", "sessionLog"]
  }
}
```

- [ ] __Step 2: Verify and commit__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool patterns/event-log.json > /dev/null && echo OK && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add patterns/event-log.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add patterns/event-log.json"
```

---

## Task B.3: Create `patterns/streak-counter.json`

__Files:__
- Create: `~/claude-personal-trainer/patterns/streak-counter.json`

- [ ] __Step 1: Write the pattern__

```json
{
  "id": "streak-counter",
  "description": "Track consecutive-day habits or streaks. Each entry is a habit with a current streak length and last-update date.",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["habit", "currentStreak", "lastUpdate"],
      "properties": {
        "habit": {"type": "string"},
        "currentStreak": {"type": "integer", "minimum": 0},
        "lastUpdate": {"type": "string", "format": "date"},
        "longestStreak": {"type": "integer", "minimum": 0},
        "since": {"type": "string", "format": "date"},
        "notes": {"type": "string"}
      },
      "additionalProperties": true
    }
  },
  "exampleBlock": [
    {"habit": "Daily achilles 0-1/10 (no pain)", "currentStreak": 14, "longestStreak": 14, "lastUpdate": "2026-05-07", "since": "2026-04-22"}
  ],
  "rendererHint": "streak-card",
  "naming": {
    "guidance": "The top-level key in data.json is conventionally 'streaks'. Individual habits are entries inside the array.",
    "examples": ["streaks"]
  }
}
```

- [ ] __Step 2: Verify and commit__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool patterns/streak-counter.json > /dev/null && echo OK && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add patterns/streak-counter.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add patterns/streak-counter.json"
```

---

## Task B.4: Create `patterns/periodic-review.json`

__Files:__
- Create: `~/claude-personal-trainer/patterns/periodic-review.json`

- [ ] __Step 1: Write the pattern__

```json
{
  "id": "periodic-review",
  "description": "Capture periodic retros (weekly, monthly, quarterly, science reviews). Each entry is a structured reflection with findings + changes-made.",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["date", "period", "findings"],
      "properties": {
        "date": {"type": "string", "format": "date"},
        "period": {"type": "string"},
        "findings": {"type": "string"},
        "changes": {"type": "string"},
        "metricsSnapshot": {"type": "object"}
      },
      "additionalProperties": true
    }
  },
  "exampleBlock": [
    {"date": "2026-05-07", "period": "Week 9 transition", "findings": "Strength progressing.", "changes": "Hold targets per Section 7."}
  ],
  "rendererHint": "timeline-table",
  "naming": {
    "guidance": "Common names: 'reviews', 'retros', 'scienceReviews'.",
    "examples": ["reviews", "retros", "scienceReviews"]
  }
}
```

- [ ] __Step 2: Verify and commit__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool patterns/periodic-review.json > /dev/null && echo OK && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add patterns/periodic-review.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add patterns/periodic-review.json"
```

---

## Task B.5: Create `patterns/tagged-collection.json`

__Files:__
- Create: `~/claude-personal-trainer/patterns/tagged-collection.json`

- [ ] __Step 1: Write the pattern__

```json
{
  "id": "tagged-collection",
  "description": "A tagged collection of items: recipes, song ideas, study cards, project ideas, references. Items have free-form content + tags for retrieval.",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["id", "title"],
      "properties": {
        "id": {"type": "string"},
        "title": {"type": "string"},
        "content": {"type": "string"},
        "tags": {"type": "array", "items": {"type": "string"}},
        "createdDate": {"type": "string", "format": "date"},
        "updatedDate": {"type": "string", "format": "date"}
      },
      "additionalProperties": true
    }
  },
  "exampleBlock": [
    {"id": "card-001", "title": "Spanish: subjunctive after esperar que", "content": "Use subjunctive mood.", "tags": ["spanish", "grammar"], "createdDate": "2026-05-07"}
  ],
  "rendererHint": "kpi-tiles",
  "naming": {
    "guidance": "Pick a name describing the collection. Use camelCase or plural noun.",
    "examples": ["recipes", "songIdeas", "studyCards"]
  }
}
```

- [ ] __Step 2: Verify and commit__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool patterns/tagged-collection.json > /dev/null && echo OK && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add patterns/tagged-collection.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add patterns/tagged-collection.json"
```

---

## Task B.6: Create `widgets/streak-card.html` (XSS-safe)

__Files:__
- Create: `~/claude-personal-trainer/widgets/streak-card.html`

All widgets use a small `esc()` HTML-escape helper before any user-controlled string touches the DOM via template strings + innerHTML. Static markup is safe to assemble; only escape user data.

- [ ] __Step 1: Create widgets directory__

```bash
mkdir -p ~/claude-personal-trainer/widgets
```

- [ ] __Step 2: Write the widget__

```html
<!-- WIDGET: streak-card -->
<!-- Renders: streak-counter pattern -->
<style>
  .streak-card-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 1rem; }
  .streak-card { padding: 1rem; border: 1px solid #ddd; border-radius: 8px; background: #fafafa; }
  .streak-card h4 { margin: 0 0 0.5rem 0; font-size: 0.9rem; color: #555; }
  .streak-card .count { font-size: 2.5rem; font-weight: bold; line-height: 1; color: #2a8b2e; }
  .streak-card .meta { font-size: 0.8rem; color: #888; margin-top: 0.5rem; }
  .streak-card.broken .count { color: #888; }
</style>

<h3>Streaks</h3>
<div class="streak-card-grid" id="streak-card-mount">Loading...</div>

<script>
(async () => {
  const esc = s => String(s == null ? '' : s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const data = await fetch('./data.json').then(r => r.json());
  const streaks = Array.isArray(data.streaks) ? data.streaks : [];
  const mount = document.getElementById('streak-card-mount');
  if (streaks.length === 0) {
    mount.textContent = 'No streaks tracked yet.';
    return;
  }
  const html = streaks.map(s => {
    const broken = Number(s.currentStreak) === 0 ? 'broken' : '';
    const habit = esc(s.habit);
    const cur = Number.isFinite(Number(s.currentStreak)) ? Number(s.currentStreak) : 0;
    const longest = Number.isFinite(Number(s.longestStreak)) ? Number(s.longestStreak) : cur;
    const last = esc(s.lastUpdate);
    return '<div class="streak-card ' + broken + '">'
      + '<h4>' + habit + '</h4>'
      + '<div class="count">' + cur + '</div>'
      + '<div class="meta">days &middot; best ' + longest + ' &middot; last ' + last + '</div>'
      + '</div>';
  }).join('');
  mount.innerHTML = html;
})();
</script>
```

- [ ] __Step 3: Smoke test in browser__

```bash
mkdir -p /tmp/widget-test && cat > /tmp/widget-test/data.json <<'EOF'
{"streaks": [{"habit": "No pain days", "currentStreak": 14, "longestStreak": 21, "lastUpdate": "2026-05-07"}, {"habit": "Daily practice", "currentStreak": 5, "longestStreak": 30, "lastUpdate": "2026-05-07"}]}
EOF
cp ~/claude-personal-trainer/widgets/streak-card.html /tmp/widget-test/index.html
cd /tmp/widget-test && python3 -m http.server 8765 &
SERVER_PID=$!
echo "Open http://localhost:8765/ — should see two streak cards: 14 days, 5 days"
echo "Kill server: kill $SERVER_PID"
```

- [ ] __Step 4: XSS smoke test (verify esc() works)__

```bash
cat > /tmp/widget-test/data.json <<'EOF'
{"streaks": [{"habit": "<script>alert('XSS')</script>", "currentStreak": 5, "longestStreak": 5, "lastUpdate": "2026-05-07"}]}
EOF
echo "Reload http://localhost:8765/ — habit text should display literally as '<script>alert('XSS')</script>'; no alert should fire"
kill $SERVER_PID
```

- [ ] __Step 5: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add widgets/streak-card.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add widgets/streak-card.html with XSS-safe esc() helper"
```

---

## Task B.7: Create `widgets/progress-chart.html`

__Files:__
- Create: `~/claude-personal-trainer/widgets/progress-chart.html`

Uses SVG for the chart. SVG element creation goes through `document.createElementNS` so user-controlled tooltip text is safely set via `textContent`, never via innerHTML.

- [ ] __Step 1: Write the widget__

```html
<!-- WIDGET: progress-chart -->
<!-- Renders: time-series-numeric pattern -->
<style>
  .progress-chart-container { padding: 1rem; border: 1px solid #ddd; border-radius: 8px; background: white; }
  .progress-chart-container h4 { margin: 0 0 0.5rem 0; }
  .progress-chart-svg { width: 100%; height: 200px; }
  .progress-chart-svg .line { fill: none; stroke: #2a8b2e; stroke-width: 2; }
  .progress-chart-svg .dot { fill: #2a8b2e; }
  .progress-chart-svg .axis { stroke: #ccc; stroke-width: 1; }
  .progress-chart-meta { font-size: 0.8rem; color: #888; margin-top: 0.5rem; }
</style>

<div class="progress-chart-container">
  <h4 id="progress-chart-title">Progress Chart</h4>
  <svg class="progress-chart-svg" id="progress-chart-svg" viewBox="0 0 600 200" preserveAspectRatio="none"></svg>
  <div class="progress-chart-meta" id="progress-chart-meta"></div>
</div>

<script>
(async () => {
  const SVG_NS = 'http://www.w3.org/2000/svg';
  const data = await fetch('./data.json').then(r => r.json());
  // Find the first array of {date, value} entries with at least 2 items
  let series = null, key = null;
  for (const k in data) {
    const v = data[k];
    if (Array.isArray(v) && v.length >= 2 && v[0] && v[0].date && v[0].value !== undefined) {
      series = v; key = k; break;
    }
  }
  const titleEl = document.getElementById('progress-chart-title');
  const svgEl = document.getElementById('progress-chart-svg');
  const metaEl = document.getElementById('progress-chart-meta');
  if (!series) {
    metaEl.textContent = 'No time-series-numeric data found.';
    return;
  }
  // Use textContent (auto-escapes) for the title
  titleEl.textContent = key + ' over time';
  const sorted = series.slice().sort((a, b) => String(a.date).localeCompare(String(b.date)));
  const values = sorted.map(e => Number(e.value));
  const min = Math.min.apply(null, values);
  const max = Math.max.apply(null, values);
  const range = (max - min) || 1;
  const W = 600, H = 200, P = 20;
  const pts = sorted.map((e, i) => {
    const x = P + (i / (sorted.length - 1)) * (W - 2*P);
    const y = H - P - ((Number(e.value) - min) / range) * (H - 2*P);
    return [x, y, e];
  });
  // Build axes
  const ax1 = document.createElementNS(SVG_NS, 'line');
  ax1.setAttribute('class', 'axis');
  ax1.setAttribute('x1', P); ax1.setAttribute('y1', H-P);
  ax1.setAttribute('x2', W-P); ax1.setAttribute('y2', H-P);
  svgEl.appendChild(ax1);
  const ax2 = document.createElementNS(SVG_NS, 'line');
  ax2.setAttribute('class', 'axis');
  ax2.setAttribute('x1', P); ax2.setAttribute('y1', P);
  ax2.setAttribute('x2', P); ax2.setAttribute('y2', H-P);
  svgEl.appendChild(ax2);
  // Build polyline path
  const pathEl = document.createElementNS(SVG_NS, 'path');
  pathEl.setAttribute('class', 'line');
  pathEl.setAttribute('d', pts.map((p, i) => (i === 0 ? 'M' : 'L') + p[0].toFixed(1) + ',' + p[1].toFixed(1)).join(' '));
  svgEl.appendChild(pathEl);
  // Dots with title (tooltip) — title content set via textContent, safe
  pts.forEach(p => {
    const c = document.createElementNS(SVG_NS, 'circle');
    c.setAttribute('class', 'dot');
    c.setAttribute('cx', p[0].toFixed(1));
    c.setAttribute('cy', p[1].toFixed(1));
    c.setAttribute('r', 3);
    const t = document.createElementNS(SVG_NS, 'title');
    t.textContent = String(p[2].date) + ': ' + String(p[2].value) + (p[2].unit ? ' ' + p[2].unit : '');
    c.appendChild(t);
    svgEl.appendChild(c);
  });
  // Meta line — use textContent (auto-escapes)
  const last = sorted[sorted.length - 1];
  const unit = last.unit ? String(last.unit) : '';
  metaEl.textContent = sorted.length + ' entries · range ' + min + '–' + max + ' ' + unit + ' · latest ' + last.value + ' on ' + last.date;
})();
</script>
```

- [ ] __Step 2: Smoke test__

```bash
cat > /tmp/widget-test/data.json <<'EOF'
{"bodyLog": [{"date": "2026-04-28", "value": 221.3, "unit": "lbs"}, {"date": "2026-05-01", "value": 220.8, "unit": "lbs"}, {"date": "2026-05-04", "value": 220.5, "unit": "lbs"}, {"date": "2026-05-07", "value": 220.2, "unit": "lbs"}]}
EOF
cp ~/claude-personal-trainer/widgets/progress-chart.html /tmp/widget-test/index.html
cd /tmp/widget-test && python3 -m http.server 8765 &
SERVER_PID=$!
echo "Open http://localhost:8765/ — should see downward-sloping line chart with 4 points"
echo "Kill: kill $SERVER_PID"
```

- [ ] __Step 3: Commit__

```bash
kill $SERVER_PID 2>/dev/null
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add widgets/progress-chart.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add widgets/progress-chart.html (SVG-based, XSS-safe via createElementNS + textContent)"
```

---

## Task B.8: Create `widgets/timeline-table.html` (XSS-safe)

__Files:__
- Create: `~/claude-personal-trainer/widgets/timeline-table.html`

- [ ] __Step 1: Write the widget__

```html
<!-- WIDGET: timeline-table -->
<!-- Renders: event-log or periodic-review pattern -->
<style>
  .timeline-table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
  .timeline-table th, .timeline-table td { padding: 0.5rem; border-bottom: 1px solid #eee; text-align: left; vertical-align: top; }
  .timeline-table th { font-size: 0.75rem; text-transform: uppercase; color: #888; }
  .timeline-table .date { white-space: nowrap; color: #555; font-family: monospace; }
  .timeline-table .activity { font-weight: 600; }
  .timeline-table .notes { color: #666; }
  .timeline-empty { font-style: italic; color: #888; padding: 1rem; }
</style>

<h3 id="timeline-title">Timeline</h3>
<div id="timeline-mount">Loading...</div>

<script>
(async () => {
  const esc = s => String(s == null ? '' : s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const data = await fetch('./data.json').then(r => r.json());
  let log = null, key = null;
  for (const k in data) {
    const v = data[k];
    if (Array.isArray(v) && v.length > 0 && v[0] && v[0].date) { log = v; key = k; break; }
  }
  const mount = document.getElementById('timeline-mount');
  const titleEl = document.getElementById('timeline-title');
  if (!log) {
    mount.innerHTML = '<div class="timeline-empty">No event log found.</div>';
    return;
  }
  titleEl.textContent = key;
  const sorted = log.slice().sort((a, b) => String(b.date).localeCompare(String(a.date))).slice(0, 30);
  const rows = sorted.map(e => {
    const dateText = esc(e.date);
    const activityText = esc(e.activity || e.type || e.period || '—');
    const notesText = esc(String(e.notes || e.findings || '').slice(0, 200));
    return '<tr>'
      + '<td class="date">' + dateText + '</td>'
      + '<td class="activity">' + activityText + '</td>'
      + '<td class="notes">' + notesText + '</td>'
      + '</tr>';
  }).join('');
  mount.innerHTML = '<table class="timeline-table">'
    + '<thead><tr><th>Date</th><th>Activity</th><th>Details</th></tr></thead>'
    + '<tbody>' + rows + '</tbody>'
    + '</table>';
})();
</script>
```

- [ ] __Step 2: Smoke test (including XSS verification)__

```bash
cat > /tmp/widget-test/data.json <<'EOF'
{"activityLog": [{"date": "2026-05-06", "type": "training", "activity": "<script>alert('xss')</script>", "notes": "TB DL 240"}, {"date": "2026-05-05", "type": "game", "activity": "Basketball", "notes": "30 min high"}]}
EOF
cp ~/claude-personal-trainer/widgets/timeline-table.html /tmp/widget-test/index.html
cd /tmp/widget-test && python3 -m http.server 8765 &
SERVER_PID=$!
echo "Open http://localhost:8765/ — should see 2-row table; the first row activity should display literal text '<script>alert('xss')</script>', no alert"
kill $SERVER_PID
```

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add widgets/timeline-table.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add widgets/timeline-table.html with XSS-safe esc() helper"
```

---

## Task B.9: Create `widgets/weekly-grid.html` (XSS-safe)

__Files:__
- Create: `~/claude-personal-trainer/widgets/weekly-grid.html`

- [ ] __Step 1: Write the widget__

```html
<!-- WIDGET: weekly-grid -->
<!-- Renders: event-log pattern grouped by week -->
<style>
  .weekly-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 0.5rem; }
  .weekly-grid .day { padding: 0.5rem; border: 1px solid #eee; border-radius: 4px; min-height: 80px; font-size: 0.8rem; }
  .weekly-grid .day .label { font-weight: 600; color: #555; font-size: 0.7rem; text-transform: uppercase; }
  .weekly-grid .day.has-event { background: #f0f8f0; border-color: #c6e2c8; }
  .weekly-grid .day .event-list { margin-top: 0.25rem; }
  .weekly-grid .day .event-item { font-size: 0.75rem; color: #444; }
</style>

<h3>This Week</h3>
<div class="weekly-grid" id="weekly-grid-mount"></div>

<script>
(async () => {
  const esc = s => String(s == null ? '' : s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const data = await fetch('./data.json').then(r => r.json());
  let log = null;
  for (const k in data) {
    const v = data[k];
    if (Array.isArray(v) && v.length > 0 && v[0] && v[0].date && (v[0].type || v[0].activity)) { log = v; break; }
  }
  const mount = document.getElementById('weekly-grid-mount');
  if (!log) { mount.textContent = 'No event data.'; return; }
  const today = new Date();
  const daysFromSunday = today.getDay();
  const sunday = new Date(today); sunday.setDate(today.getDate() - daysFromSunday);
  const week = [];
  for (let i = 0; i < 7; i++) {
    const d = new Date(sunday); d.setDate(sunday.getDate() + i);
    const iso = d.toISOString().slice(0, 10);
    const events = log.filter(e => e.date === iso);
    week.push({ iso, label: d.toLocaleDateString(undefined, {weekday: 'short'}), dayOfMonth: d.getDate(), events });
  }
  const html = week.map(d => {
    const cls = d.events.length ? 'day has-event' : 'day';
    const labelText = esc(d.label) + ' ' + d.dayOfMonth;
    const eventsHtml = d.events.map(e => '<div class="event-item">' + esc(e.activity || e.type) + '</div>').join('');
    return '<div class="' + cls + '">'
      + '<div class="label">' + labelText + '</div>'
      + '<div class="event-list">' + eventsHtml + '</div>'
      + '</div>';
  }).join('');
  mount.innerHTML = html;
})();
</script>
```

- [ ] __Step 2: Smoke test__: copy to `/tmp/widget-test/index.html` and verify a 7-day grid renders, with XSS test similar to Task B.8

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add widgets/weekly-grid.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add widgets/weekly-grid.html with XSS-safe esc() helper"
```

---

## Task B.10: Create `widgets/kpi-tiles.html` (XSS-safe)

__Files:__
- Create: `~/claude-personal-trainer/widgets/kpi-tiles.html`

- [ ] __Step 1: Write the widget__

```html
<!-- WIDGET: kpi-tiles -->
<!-- Renders: any data — produces simple count/latest tiles for each top-level key in data.json -->
<style>
  .kpi-tiles { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 1rem; }
  .kpi-tile { padding: 1rem; border: 1px solid #ddd; border-radius: 8px; background: white; }
  .kpi-tile .label { font-size: 0.75rem; text-transform: uppercase; color: #888; }
  .kpi-tile .value { font-size: 2rem; font-weight: bold; color: #333; }
  .kpi-tile .unit { font-size: 0.85rem; color: #666; }
</style>

<div class="kpi-tiles" id="kpi-tiles-mount"></div>

<script>
(async () => {
  const esc = s => String(s == null ? '' : s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  const data = await fetch('./data.json').then(r => r.json());
  const mount = document.getElementById('kpi-tiles-mount');
  const tiles = [];
  for (const k in data) {
    const v = data[k];
    if (Array.isArray(v)) {
      if (v.length === 0) continue;
      const last = v[v.length - 1];
      if (last && last.value !== undefined) {
        tiles.push({label: k + ' (latest)', value: last.value, unit: last.unit || ''});
      } else {
        tiles.push({label: k + ' (count)', value: v.length, unit: 'entries'});
      }
    }
  }
  if (tiles.length === 0) { mount.textContent = 'No data to summarize yet.'; return; }
  const html = tiles.map(t => {
    return '<div class="kpi-tile">'
      + '<div class="label">' + esc(t.label) + '</div>'
      + '<div class="value">' + esc(t.value) + ' <span class="unit">' + esc(t.unit) + '</span></div>'
      + '</div>';
  }).join('');
  mount.innerHTML = html;
})();
</script>
```

- [ ] __Step 2: Smoke test__: verify tiles render for a multi-key data.json including XSS check

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add widgets/kpi-tiles.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add widgets/kpi-tiles.html with XSS-safe esc() helper"
```

---

## Task B.11: Create `site-templates/dashboard-shell.html`

__Files:__
- Create: `~/claude-personal-trainer/site-templates/dashboard-shell.html`

- [ ] __Step 1: Create directory and write the shell__

```bash
mkdir -p ~/claude-personal-trainer/site-templates
```

- [ ] __Step 2: Write the shell__ (note: this is template-time content, not user-runtime — placeholders are replaced by the wizard during generation, not by client-side JS)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{DOMAIN_TITLE}}</title>
  <style>
    :root {
      --bg: #fafafa; --surface: #fff; --text: #222; --text-muted: #666;
      --accent: #2a8b2e; --border: #e0e0e0;
    }
    body { margin: 0; font-family: system-ui, -apple-system, sans-serif; background: var(--bg); color: var(--text); }
    header { padding: 1.5rem 2rem; background: var(--surface); border-bottom: 1px solid var(--border); }
    header h1 { margin: 0; font-size: 1.5rem; }
    header .subtitle { color: var(--text-muted); font-size: 0.9rem; margin-top: 0.25rem; }
    main { max-width: 1200px; margin: 0 auto; padding: 2rem; }
    .tabs { display: flex; gap: 1rem; margin-bottom: 2rem; border-bottom: 1px solid var(--border); }
    .tabs button { background: none; border: none; padding: 0.75rem 1rem; font-size: 0.9rem; color: var(--text-muted); cursor: pointer; border-bottom: 2px solid transparent; }
    .tabs button.active { color: var(--accent); border-bottom-color: var(--accent); }
    .tab-content { display: none; }
    .tab-content.active { display: block; }
    section { margin-bottom: 2rem; }
    h2, h3 { color: var(--text); }
  </style>
</head>
<body>
  <header>
    <h1>{{DOMAIN_TITLE}}</h1>
    <div class="subtitle">{{USER_TAGLINE}}</div>
  </header>
  <main>
    <div class="tabs">
      <button class="active" data-tab="today">Today</button>
      <button data-tab="history">History</button>
      <button data-tab="knowledge">Knowledge</button>
    </div>

    <div id="tab-today" class="tab-content active">
      <!-- WIDGET-MOUNT: today -->
      {{TODAY_WIDGETS}}
    </div>

    <div id="tab-history" class="tab-content">
      <!-- WIDGET-MOUNT: history -->
      {{HISTORY_WIDGETS}}
    </div>

    <div id="tab-knowledge" class="tab-content">
      <h2>Knowledge</h2>
      <p>See the <code>knowledge/</code> folder in this repo for evidence-synthesized modules.</p>
    </div>
  </main>

  <script>
    document.querySelectorAll('.tabs button').forEach(btn => {
      btn.addEventListener('click', () => {
        document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.tabs button').forEach(b => b.classList.remove('active'));
        const target = btn.getAttribute('data-tab');
        document.getElementById('tab-' + target).classList.add('active');
        btn.classList.add('active');
      });
    });
  </script>
</body>
</html>
```

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add site-templates/dashboard-shell.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add site-templates/dashboard-shell.html"
```

---

## Task B.12: Create `site-templates/log-viewer-shell.html`

__Files:__
- Create: `~/claude-personal-trainer/site-templates/log-viewer-shell.html`

- [ ] __Step 1: Write the shell__

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>{{DOMAIN_TITLE}}</title>
  <style>
    body { margin: 0; font-family: system-ui, sans-serif; max-width: 800px; margin: 0 auto; padding: 2rem; color: #222; }
    header h1 { margin: 0 0 0.5rem 0; }
    header .subtitle { color: #666; }
    section { margin: 2rem 0; }
  </style>
</head>
<body>
  <header>
    <h1>{{DOMAIN_TITLE}}</h1>
    <div class="subtitle">{{USER_TAGLINE}}</div>
  </header>

  <main>
    {{LOG_WIDGETS}}
  </main>
</body>
</html>
```

- [ ] __Step 2: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add site-templates/log-viewer-shell.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add site-templates/log-viewer-shell.html"
```

---

## Task B.13: End-to-end smoke test of patterns + widgets

__Files:__
- No file changes — verification only

- [ ] __Step 1: Hand-compose synthetic data.json combining all 5 patterns__

```bash
mkdir -p /tmp/cpt-smoke && cat > /tmp/cpt-smoke/data.json <<'EOF'
{
  "meta": {"domain": "Test", "created_date": "2026-05-07"},
  "bodyLog": [
    {"date": "2026-04-28", "value": 221.3, "unit": "lbs"},
    {"date": "2026-05-01", "value": 220.8, "unit": "lbs"},
    {"date": "2026-05-04", "value": 220.5, "unit": "lbs"},
    {"date": "2026-05-07", "value": 220.2, "unit": "lbs"}
  ],
  "activityLog": [
    {"date": "2026-05-06", "type": "training", "activity": "Lower Body", "notes": "TB DL 240"},
    {"date": "2026-05-05", "type": "game", "activity": "Basketball", "notes": "30 min high"}
  ],
  "streaks": [
    {"habit": "No-pain days", "currentStreak": 14, "longestStreak": 21, "lastUpdate": "2026-05-07"}
  ],
  "reviews": [
    {"date": "2026-05-04", "period": "Week 9", "findings": "Recomp on track.", "changes": "Phase 2 plan defined."}
  ],
  "studyCards": [
    {"id": "c-001", "title": "Test card", "tags": ["test"], "createdDate": "2026-05-07"}
  ]
}
EOF
```

- [ ] __Step 2: Compose hand-built index.html using all 5 widgets__

```bash
cat > /tmp/cpt-smoke/index.html <<'EOF'
<!DOCTYPE html><html><head><title>CPT Smoke Test</title></head><body><main>
EOF
for w in streak-card progress-chart timeline-table weekly-grid kpi-tiles; do
  echo "<section>" >> /tmp/cpt-smoke/index.html
  cat ~/claude-personal-trainer/widgets/$w.html >> /tmp/cpt-smoke/index.html
  echo "</section>" >> /tmp/cpt-smoke/index.html
done
echo "</main></body></html>" >> /tmp/cpt-smoke/index.html
```

- [ ] __Step 3: Serve and verify in browser__

```bash
cd /tmp/cpt-smoke && python3 -m http.server 8765 &
SERVER_PID=$!
echo "Open http://localhost:8765/ — verify all 5 widgets render with sensible data"
echo "Each widget: streak-card = 14 days; progress-chart = 4-pt downward line; timeline-table = 2 rows; weekly-grid = current week; kpi-tiles = tiles per array"
echo "Kill: kill $SERVER_PID"
```

- [ ] __Step 4: Stop server__

```bash
kill $SERVER_PID
```

If everything renders correctly, Phase B is complete. If not, iterate on the failing widget(s).

---

# PHASE C: Knowledge Bank Seed

## Task C.1: Create `knowledge-bank/_index.json` with controlled vocabulary

__Files:__
- Create: `~/claude-personal-trainer/knowledge-bank/_index.json`

- [ ] __Step 1: Create directory and write the index__

```bash
mkdir -p ~/claude-personal-trainer/knowledge-bank
```

- [ ] __Step 2: Write the index__

```json
{
  "controlledVocabulary": {
    "strength": "Resistance training, hypertrophy, strength adaptations, periodization",
    "tendinopathy": "Tendon injury management, loading protocols, return-to-play",
    "recovery": "Sleep, fatigue, deloads, recovery modalities",
    "nutrition": "Macros, meal timing, fueling for sport, body composition",
    "body-composition": "Measurement methods, recomp protocols, lean-mass preservation",
    "in-season-athlete": "Athletes balancing competition with training",
    "evidence-discipline": "How to find and verify sources; meta-rules for citation"
  },
  "modules": {}
}
```

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add knowledge-bank/_index.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add knowledge-bank/_index.json scaffold"
```

---

## Task C.2: Harvest + de-personalize achilles HSR module

__Files:__
- Create: `~/claude-personal-trainer/knowledge-bank/tendinopathy/achilles-hsr-protocol.md`
- Modify: `~/claude-personal-trainer/knowledge-bank/_index.json`

- [ ] __Step 1: Read source and copy with de-personalization__

```bash
mkdir -p ~/claude-personal-trainer/knowledge-bank/tendinopathy
cp ~/training-program/knowledge/achilles-tendinopathy-hsr.md ~/claude-personal-trainer/knowledge-bank/tendinopathy/achilles-hsr-protocol.md
```

- [ ] __Step 2: Open in editor and remove all personal references__

Manually scan and remove:
- Any "Menotti", "user", or athlete-specific name references
- Any specific body-weight, age, or injury-history numbers (keep generic protocol description)
- Any references to specific dates or weeks of "the program"
- Any "we" or "you" framings that imply a specific user; replace with "the athlete" or "the lifter"

The synthesized protocol content (Beyer 2015 dosing, pain monitoring rules, etc.) is the keep-value; trim everything else.

- [ ] __Step 3: Add a generic "How to apply" section__

```markdown
## How to apply

This module describes the Beyer 2015 HSR protocol generically. For an athlete with achilles tendinopathy:
- Choose between bilateral (seated, soleus-bias) and single-leg (standing, gastroc-bias) variants based on phase / coach guidance
- Follow the 12-week load progression (15RM to 6RM)
- Apply pain-monitoring thresholds (0-3/10 acceptable, 4-5/10 hold, >5/10 stop and reduce)
- Tendons adapt slowly; expect symptom improvement at 6-12 weeks, structural change at 3-6 months
```

- [ ] __Step 4: Update `_index.json` to register the module__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import json, pathlib
p = pathlib.Path('knowledge-bank/_index.json')
idx = json.loads(p.read_text())
idx['modules']['tendinopathy/achilles-hsr-protocol.md'] = ['tendinopathy', 'in-season-athlete', 'recovery']
p.write_text(json.dumps(idx, indent=2) + '\n')
print('Updated _index.json')
EOF
```

- [ ] __Step 5: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add knowledge-bank/tendinopathy/achilles-hsr-protocol.md knowledge-bank/_index.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add knowledge-bank/tendinopathy/achilles-hsr-protocol.md (de-personalized)"
```

---

## Task C.3: Harvest + de-personalize remaining knowledge modules

Apply the same harvest-and-de-personalize procedure as Task C.2 for each of these source files. Each subtask = a separate commit.

| Source | Destination | Tags |
|---|---|---|
| `~/training-program/knowledge/in-season-volume.md` | `knowledge-bank/strength/in-season-volume.md` | `strength`, `in-season-athlete` |
| `~/training-program/knowledge/protein-distribution.md` | `knowledge-bank/nutrition/protein-distribution.md` | `nutrition` |
| `~/training-program/knowledge/recovery-sleep.md` | `knowledge-bank/recovery/sleep-and-performance.md` | `recovery`, `evidence-discipline` |
| `~/training-program/knowledge/body-composition-measurement.md` | `knowledge-bank/body-composition/measurement-methods.md` | `body-composition`, `nutrition` |
| `~/training-program/knowledge/pre-game-fueling.md` | `knowledge-bank/nutrition/pre-game-fueling.md` | `nutrition`, `in-season-athlete` |
| `~/training-program/knowledge/sodium-hydration.md` | `knowledge-bank/nutrition/sodium-hydration.md` | `nutrition`, `in-season-athlete` |
| `~/training-program/knowledge/supplements.md` | `knowledge-bank/recovery/supplements.md` | `recovery`, `nutrition` |
| `~/training-program/knowledge/hormones-testosterone.md` | `knowledge-bank/recovery/hormones-and-recovery.md` | `recovery` |

- [ ] __Step 1: Run all 8 harvests in sequence__ (for each row above, follow steps 1-5 from Task C.2; one commit per harvest)

- [ ] __Step 2: After all 8 are complete, verify `_index.json`__

```bash
cd ~/claude-personal-trainer && python3 -c "import json; idx=json.load(open('knowledge-bank/_index.json')); print(f'Modules registered: {len(idx[\"modules\"])}'); [print(f' - {k}: {v}') for k, v in idx['modules'].items()]"
```
Expected: 9 modules registered (achilles + 8 from this task)

---

## Task C.4: Author new module — `strength/progressive-overload.md`

This module is synthesized from Section 1 of the existing CLAUDE.md (Loading and Progression Rules), generalized.

__Files:__
- Create: `~/claude-personal-trainer/knowledge-bank/strength/progressive-overload.md`
- Modify: `~/claude-personal-trainer/knowledge-bank/_index.json`

- [ ] __Step 1: Write the module__

```bash
mkdir -p ~/claude-personal-trainer/knowledge-bank/strength
```

```markdown
# Progressive Overload — Practical Programming

Progressive overload is the principle that for continued strength or hypertrophy adaptation, training stimulus must increase over time. The simplest, most reliable model for intermediate lifters is __double progression__: complete all prescribed reps with good form across two consecutive sessions at the same weight, then increase load.

## Key findings

- __Double progression is the most reliable progression method for intermediate lifters training 2-3x/week.__ Source: Schoenfeld et al. (2017) "Dose-response relationship between weekly resistance training volume and increases in muscle mass" (PMID 27433992); NSCA Essentials of Strength Training (4th ed.) Ch. 18. Evidence tier: __Strong__.
- __Load increment per cycle:__ compound lower body 10 lbs, compound upper body 5 lbs, dumbbell accessories 5 lbs, machines/cables one plate increment, bodyweight exercises add reps first then external load 5-10 lbs. Evidence tier: __Strong__ (NSCA convention).
- __When stalled:__ check recovery factors first, then microload (2.5 lb fractional plates), then add a set at current weight, then last-resort 10% deload + rebuild. Source: NSCA + Pritchard et al. 2015 deload review. Evidence tier: __Moderate__.
- __RPE targets shift by phase:__ Foundation phases use RPE 6-7 (3-4 reps in reserve), Strength-Power phases use RPE 7-8 (2-3 RIR), Power Realization peaks at RPE 7-9 on primary lifts. Source: Helms et al. 2018 RIR-based programming literature; NSCA periodization conventions. Evidence tier: __Strong__.

## Practical guidance

- Pick a starting weight at RPE 6-7 for the prescribed reps. Don't go heavier on day 1.
- Track every set. Without records, "did I hit 4x5 at this weight cleanly last time?" becomes a guess.
- A stall is more often a recovery problem than a load problem. Audit sleep, nutrition, and total weekly volume before reducing weight.
- Microloading (2.5 lb fractional plates) is the most under-used progression tool. Especially valuable for upper-body lifts where 5 lb jumps are aggressive at intermediate level.
- Don't substitute set-and-rep schemes mid-program. If the plan is 4x5, do 4x5; switching to 3x10 changes the stimulus and breaks progression tracking.

## Corrections log

(empty)

## References

- Schoenfeld et al. (2017). Dose-response relationship between weekly resistance training volume and increases in muscle mass. PMID 27433992.
- Helms et al. (2018). RIR-based RPE in resistance training. Journal of Strength and Conditioning Research.
- Pritchard et al. (2015). Effects and mechanisms of tapering in maximizing muscular strength. Strength and Conditioning Journal.
- NSCA Essentials of Strength Training (4th ed.) Ch. 18 Program Design.

## How to apply

For any strength practice (weightlifting, climbing, gymnastics, even sport-specific drills with measurable load):
- Default to double progression
- Microload before reducing
- Treat stalls as recovery questions before they're load questions
- Calibrate RPE expectations to the program phase, not the individual session
```

- [ ] __Step 2: Register and commit__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import json, pathlib
p = pathlib.Path('knowledge-bank/_index.json')
idx = json.loads(p.read_text())
idx['modules']['strength/progressive-overload.md'] = ['strength']
p.write_text(json.dumps(idx, indent=2) + '\n')
EOF
git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  add knowledge-bank/strength/progressive-overload.md knowledge-bank/_index.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  commit -m "Add knowledge-bank/strength/progressive-overload.md"
```

---

## Task C.5: Author new module — `strength/deload-protocols.md`

Synthesized from Section 2 of the existing CLAUDE.md.

__Files:__
- Create: `~/claude-personal-trainer/knowledge-bank/strength/deload-protocols.md`
- Modify: `~/claude-personal-trainer/knowledge-bank/_index.json`

- [ ] __Step 1: Write the module__

```markdown
# Deload Protocols — Reactive vs. Scheduled

Coleman et al. 2024 (PMID 38274324) established that __scheduled prophylactic deloads in non-fatigued lifters reduce strength gains__ without compensating hypertrophy or endurance benefit. Reactive deloads triggered by measurable fatigue remain evidence-supported.

## Key findings

- __Scheduled-calendar deloads are not supported in lifters who aren't accumulating fatigue.__ Source: Coleman et al. 2024 (PMID 38274324). Evidence tier: __Strong__ (RCT, n=39).
- __Reactive-deload triggers (any 2+ warrant a 4-7 day volume cut):__ RPE on working sets 1-2 points higher than prior week at same load; performance drops >5% at matched RPE; sleep declining 3+ consecutive nights; resting HR elevated >5 bpm above norm for 3+ days. Source: Meeusen et al. 2013 overtraining consensus (PMID 23247672). Evidence tier: __Moderate__.
- __How to deload:__ reduce volume 40-50% (cut total sets by half), maintain intensity at 85-95% of working weights. Source: Pritchard et al. 2015. Evidence tier: __Strong__.
- __Tendon-load exception:__ if HSR (heavy slow resistance) is part of the program, hold full HSR load through deloads. Tendons have months-long adaptation cycles and don't benefit from reduced load the way muscles do. Source: Magnusson and Kjaer 2019 (PMC6395417). Evidence tier: __Moderate__.

## Practical guidance

- Don't take scheduled deloads as gospel. If you're hitting RPE targets, sleeping well, and progressing, skip and continue.
- Conversely, take reactive deloads seriously. Pushing through 3 consecutive sub-6h sleep nights is how injury or burnout happens.
- During a deload, keep the same exercises and weights. Just cut sets. Don't abandon the program.
- HSR / heavy tendon loading is the exception. Hold full load there.
- Pre-competition tapers are different from reactive deloads: ~14 days, volume cut 41-60% (Bosquet et al. 2007 PMID 17762369), produce small performance gains. Use only if you have a named competition.

## References

- Coleman et al. (2024). Effects of a one-week prophylactic deload on resistance training performance. PeerJ. PMID 38274324.
- Bosquet et al. (2007). Effects of tapering on performance: a meta-analysis. Med Sci Sports Exerc. PMID 17762369.
- Pritchard et al. (2015). Effects and mechanisms of tapering. Strength and Conditioning Journal.
- Magnusson and Kjaer (2019). The pathogenesis of tendinopathy. PMC6395417.
- Meeusen et al. (2013). Prevention, diagnosis, and treatment of overtraining syndrome. ECSS/ACSM consensus. PMID 23247672.

## How to apply

For any strength practice with periodic high-fatigue accumulation:
- Use reactive deloads (trigger-based), not scheduled-calendar deloads
- During a deload, cut volume not intensity
- Preserve heavy-tendon loading even during deloads if applicable
```

- [ ] __Step 2: Register and commit__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import json, pathlib
p = pathlib.Path('knowledge-bank/_index.json')
idx = json.loads(p.read_text())
idx['modules']['strength/deload-protocols.md'] = ['strength', 'recovery']
p.write_text(json.dumps(idx, indent=2) + '\n')
EOF
git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  add knowledge-bank/strength/deload-protocols.md knowledge-bank/_index.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  commit -m "Add knowledge-bank/strength/deload-protocols.md"
```

---

## Task C.6: Author meta-module — `general-evidence-discipline/how-to-cite.md`

This module is __always__ included in every user's seed knowledge regardless of domain.

__Files:__
- Create: `~/claude-personal-trainer/knowledge-bank/general-evidence-discipline/how-to-cite.md`
- Modify: `~/claude-personal-trainer/knowledge-bank/_index.json`

- [ ] __Step 1: Write the module__

```bash
mkdir -p ~/claude-personal-trainer/knowledge-bank/general-evidence-discipline
```

```markdown
# How to Cite — Evidence Discipline

This is a meta-module: rules for finding, verifying, and citing sources when writing knowledge files in any domain. Always included in `claude-personal-trainer` seed knowledge regardless of the user's domain.

## The three-step protocol

When making any claim that warrants a source:

1. __Check the existing `knowledge/` folder first.__ If the topic is already covered, cite from the verified sources listed there. The knowledge base is the default source of truth.
2. __If the topic isn't in `knowledge/` (or the current file is incomplete):__ search the web for trusted sources. For scientific domains: PubMed, journal sites, systematic reviews, position stands from governing bodies. For non-scientific domains: primary sources (e.g., government data for finance; original lyrics/sheet music for songwriting; CSLB / state codes for construction). __Confirm each source exists and matches the claim__ by reading at least the abstract / primary content.
3. __If steps 1 and 2 find no support:__ do not hallucinate. Say so explicitly. "I can't find peer-reviewed support for this claim, so I'm not going to assert it." Offer mechanism-level reasoning if applicable, labeled clearly as speculation, not evidence.

## Citation format conventions

- For scientific claims: include author, year, journal, and ideally PubMed ID or DOI
- For data sources: include the URL or system name + retrieval date
- For governance / standards: include the standard name + version + section
- Never write "studies show" or "research suggests" without a specific source

## Evidence tiers

Label every recommendation with one of three tiers:

- __Strong:__ multiple RCTs or meta-analyses, plus textbook consensus or governing-body position stand
- __Moderate:__ mechanism-plausible with supporting studies, but mixed or small-N data
- __Emerging:__ early research, popular among practitioners but not yet consensus. __Flag clearly; do not lead with these.__

## What NOT to cite

- Influencer / podcast claims without underlying peer-reviewed support
- "Studies show" / "research suggests" without a specific named source
- Claims from supplement or product marketing
- Wikipedia as a primary source (cite the underlying primary sources Wikipedia references)
- Your own prior conversation as a "source" — internal consistency is not evidence

## Corrections discipline

When new evidence supersedes a prior claim:
- __Add a Corrections Log entry__ to the affected `knowledge/` file
- Do NOT silently revise. The record of "what was believed when" matters; revisions hide your reasoning history.

Format:

```
## Corrections log

- 2026-05-07: Original claim "X causes Y" replaced with "X correlates with Y; mechanism unclear."
  Reason: re-read of Smith et al. 2024 abstract revealed the original was a meta-analysis,
  not a causal RCT. Source: PMID 38000000.
```

## When the user asks for evidence

If the user asks "what's the source for that?", they're holding you accountable. Either:
- Give the specific source (author, year, journal, PMID/DOI)
- Or admit you don't have one and revise the claim

This is a feature of the discipline, not a failure mode. A user holding you to verifiable sources is the user using this system correctly.
```

- [ ] __Step 2: Register and commit__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import json, pathlib
p = pathlib.Path('knowledge-bank/_index.json')
idx = json.loads(p.read_text())
idx['modules']['general-evidence-discipline/how-to-cite.md'] = ['evidence-discipline']
p.write_text(json.dumps(idx, indent=2) + '\n')
EOF
git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  add knowledge-bank/general-evidence-discipline/how-to-cite.md knowledge-bank/_index.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
  commit -m "Add knowledge-bank/general-evidence-discipline/how-to-cite.md (always-included meta-module)"
```

---

## Task C.7: Verify knowledge bank state

- [ ] __Step 1: Tree the directory__

```bash
cd ~/claude-personal-trainer && find knowledge-bank/ -type f | sort
```
Expected: `_index.json` + ~12 markdown modules across `tendinopathy/`, `strength/`, `nutrition/`, `recovery/`, `body-composition/`, `general-evidence-discipline/`

- [ ] __Step 2: Verify _index.json is consistent__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import json, pathlib
idx = json.loads(pathlib.Path('knowledge-bank/_index.json').read_text())
files_in_index = set(idx['modules'].keys())
files_on_disk = {str(p.relative_to('knowledge-bank/')) for p in pathlib.Path('knowledge-bank/').rglob('*.md')}
missing_from_index = files_on_disk - files_in_index
extra_in_index = files_in_index - files_on_disk
print(f'Files on disk: {len(files_on_disk)}')
print(f'Files in index: {len(files_in_index)}')
if missing_from_index: print(f'Missing from index: {missing_from_index}')
if extra_in_index: print(f'Extra in index: {extra_in_index}')
if not missing_from_index and not extra_in_index: print('CONSISTENT')
EOF
```
Expected: `CONSISTENT`

- [ ] __Step 3: If inconsistent, fix `_index.json` and commit__

If discrepancies exist, fix manually and commit with message "Fix knowledge-bank/_index.json consistency".

---

# PHASE D: Wizard End-to-End Verification

Phase D doesn't add new code; it verifies the wizard works by running it for real and iterating on `wizard/*.md` based on what breaks.

## Task D.1: Wizard run for fictional language-learning user

__Files:__
- May modify: `wizard/interview.md`, `wizard/generation-rules.md`, `wizard/claude-md-template.md`, `wizard/archive-rules.md` based on findings

- [ ] __Step 1: Make a clean test copy__

```bash
cp -r ~/claude-personal-trainer /tmp/cpt-test-language && cd /tmp/cpt-test-language
```

- [ ] __Step 2: Open in Claude Code__

```bash
cd /tmp/cpt-test-language && claude
```

- [ ] __Step 3: As fictional user, run through wizard__

Adopt persona: "I'm learning Spanish; B1 level; goal is reading literature and conversing on trip to Madrid in 6 months. I have ~30 min/day. Want a daily session log + vocab-size tracker + weekly reviews. Voice: dry, analytical, no motivational fluff. Strict evidence (link papers on language acquisition). Local-only site, private repo."

Run through all 5 phases.

- [ ] __Step 4: After wizard completes, evaluate__

Verify:
- `data.json` parses, contains 3 patterns: `practiceLog` (event-log), `vocabSizeLog` (time-series-numeric), `reviews` (periodic-review)
- `knowledge/` contains the meta-module (`how-to-cite.md`) and any language-learning-relevant modules (likely none in v1, should be reported as empty knowledge with stub README)
- `CLAUDE.md` is well-formed, no unfilled placeholders, voice matches "dry, analytical"
- `index.html` renders the 3 widgets matching chosen patterns
- `.claude-personal-trainer/` exists with framework files

- [ ] __Step 5: Note + fix any issues__

Likely findings:
- Wizard may stumble on a non-fitness domain because prompts were drafted with fitness in mind
- Generation rules may produce odd `data.json` keys
- Knowledge module count for non-fitness domain will be just the meta-module (correct)

Document each issue and fix in `wizard/*.md` files. Commit fixes back to `~/claude-personal-trainer/` (canonical repo, not the test copy).

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/ && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Fix wizard issues found in language-learning end-to-end test"
```

- [ ] __Step 6: Re-run on fresh copy until clean__

Repeat steps 1-5 until the wizard produces a fully usable language-learning repo without manual intervention.

---

## Task D.2: Wizard run for strength-and-conditioning user

__Files:__
- May modify: `wizard/*` based on findings

- [ ] __Step 1: Fresh test copy__

```bash
rm -rf /tmp/cpt-test-strength && cp -r ~/claude-personal-trainer /tmp/cpt-test-strength && cd /tmp/cpt-test-strength
```

- [ ] __Step 2: Run wizard adopting Menotti's profile__

Persona: "40 yo male, 6'2 220lb, in-season recreational athlete (basketball Tue+Sun, hockey Sun PM), bilateral achilles tendinopathy. Goals: injury prevention > strength/power > vertical jump. Want activityLog + bodyLog + scienceReviews + streaks (achilles green-streak). Voice: direct CSCS coach. Strict peer-review evidence. Public site GitHub Pages, public repo OK (lift logs)."

- [ ] __Step 3: Verify output approximates current `training-program/`__

Compare structure:
- `data.json` has `bodyLog`, `activityLog`, `scienceReviews`, `streaks` (similar to current repo's keys)
- `CLAUDE.md` has CSCS persona, evidence-tier discipline, in-season-athlete context
- `knowledge/` has all 9-10 fitness-relevant seed modules
- `index.html` renders 4 widgets: timeline-table (activityLog), progress-chart (bodyLog), streak-card, kpi-tiles

- [ ] __Step 4: Note + fix issues; commit fixes__

Particularly look for:
- Domain-specific `data.json` field naming
- Whether seed knowledge bank coverage feels complete for fitness
- Whether wizard handles "in-season athlete" context well

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add wizard/ && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Fix wizard issues found in strength-and-conditioning end-to-end test"
```

---

## Task D.3: Verify pause/resume via `.wizard-state.json`

- [ ] __Step 1: Fresh test, complete Phase 1, exit Claude Code__

```bash
rm -rf /tmp/cpt-test-resume && cp -r ~/claude-personal-trainer /tmp/cpt-test-resume && cd /tmp/cpt-test-resume
claude  # complete Phase 1 only, exit
```

- [ ] __Step 2: Verify `.wizard-state.json` exists with Phase 1 state__

```bash
cat /tmp/cpt-test-resume/.wizard-state.json
```
Expected: JSON with `phase: 1` (or similar) plus captured Phase 1 answers

- [ ] __Step 3: Re-open Claude Code in same dir; verify wizard offers to resume__

```bash
cd /tmp/cpt-test-resume && claude
```
Expected: Claude detects `.wizard-state.json`, summarizes Phase 1, asks "resume from Phase 2 or restart?"

- [ ] __Step 4: If pause/resume doesn't work, fix `wizard/interview.md` or `CLAUDE.md` and commit__

---

## Task D.4: Verify abort handling

- [ ] __Step 1: Fresh test, start wizard, request abort mid-interview__

- [ ] __Step 2: Verify `.wizard-state.json` updated with `status: "aborted"` and no other files written__

- [ ] __Step 3: Verify on next `claude` open, wizard offers to restart cleanly (deletes state file)__

- [ ] __Step 4: Fix any issues and commit__

---

# PHASE E: Bundled Example

## Task E.1: Copy and de-personalize current `training-program/CLAUDE.md`

__Files:__
- Create: `~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/CLAUDE.md`

- [ ] __Step 1: Copy and stage for editing__

```bash
mkdir -p ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete
cp ~/training-program/CLAUDE.md ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/CLAUDE.md
```

- [ ] __Step 2: Replace personal identifiers with placeholders__

In the copied file, replace:
- `Menotti` -> `{{ATHLETE_NAME}}`
- `40-year-old` -> `{{ATHLETE_AGE}}`
- `6'2"` -> `{{ATHLETE_HEIGHT}}`
- `~220 lbs` -> `{{ATHLETE_WEIGHT}}`
- `bilateral achilles tendinopathy` -> `{{ATHLETE_INJURIES}}`
- Sport schedule references -> `{{ATHLETE_SPORT_SCHEDULE}}`
- Work-schedule references -> `{{ATHLETE_WORK_SCHEDULE}}`
- Specific weights, dates, plateau-break narratives, body-comp percentages -> remove or replace with `{{...}}`
- Live site URL -> `{{LIVE_SITE_URL}}`
- Specific week references (Wk 7 plateau, etc.) -> remove

- [ ] __Step 3: Add explanatory header__

```markdown
<!-- 
This is an EXAMPLE: a fully-grown reference repo demonstrating what a complete 
claude-personal-trainer setup looks like for an in-season recreational athlete
with achilles tendinopathy.

Placeholders ({{ATHLETE_NAME}}, etc.) show where the wizard would inject
user-specific values. In a real setup the wizard fills these in from interview answers.

The synthesized scientific guidance below (HSR protocol, in-season volume rules,
deload triggers, etc.) is the real value of this example: even if your domain
isn't strength training, this shows what evidence-discipline and structured 
coaching guidance look like in practice.
-->

# {{ATHLETE_NAME}} — Strength & Conditioning Coaching
```

(Then the rest of the de-personalized content.)

- [ ] __Step 4: Verify no real personal data remains__

```bash
cd ~/claude-personal-trainer && grep -i 'menotti\|2026-04-2\|2026-04-3\|2026-05-' examples/strength-and-conditioning-recreational-athlete/CLAUDE.md
```
Expected: NO matches (besides any in the comment header above)

- [ ] __Step 5: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/strength-and-conditioning-recreational-athlete/CLAUDE.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add example: de-personalized strength-and-conditioning CLAUDE.md"
```

---

## Task E.2: Create example `data.example.json`

__Files:__
- Create: `~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/data.example.json`

- [ ] __Step 1: Write schema-only example with synthetic entries__

```json
{
  "meta": {
    "domain": "Strength and Conditioning — Recreational Athlete",
    "created_date": "2026-05-07",
    "tracker_version": "claude-personal-trainer-v1"
  },
  "athlete": {
    "name": "{{ATHLETE_NAME}}",
    "age": "{{ATHLETE_AGE}}",
    "height": "{{ATHLETE_HEIGHT}}",
    "weight": "{{ATHLETE_WEIGHT}}",
    "injuries": "{{ATHLETE_INJURIES}}",
    "sportSchedule": "{{ATHLETE_SPORT_SCHEDULE}}",
    "goalsPriority": ["injury-prevention", "strength-power", "vertical-jump"]
  },
  "bodyLog": [
    {"date": "2026-05-01", "weight": 220, "bodyFatPct": 23, "notes": "AM weigh-in (synthetic example)"},
    {"date": "2026-05-04", "weight": 219.5, "bodyFatPct": 22.8, "notes": "(synthetic example)"}
  ],
  "activityLog": [
    {
      "date": "2026-05-01",
      "type": "training",
      "activity": "Lower Body (synthetic example)",
      "exercises": [
        {"name": "Trap Bar Deadlift", "sets": "4x5", "weight": "225 lbs"},
        {"name": "Back Squat", "sets": "3x5", "weight": "145 lbs"}
      ],
      "notes": "Synthetic example entry"
    },
    {
      "date": "2026-05-03",
      "type": "game",
      "activity": "Basketball",
      "duration": "40 min",
      "intensity": "high",
      "notes": "Synthetic example entry"
    }
  ],
  "scienceReviews": [
    {
      "date": "2026-05-04",
      "period": "Week N — synthetic example review",
      "findings": "This is a synthetic example. In a real setup, the science review captures multi-paragraph synthesis of recent training data against evidence-based guidelines.",
      "changes": "What was adjusted as a result."
    }
  ],
  "streaks": [
    {"habit": "Achilles 0-3/10 days (no flare)", "currentStreak": 7, "longestStreak": 14, "lastUpdate": "2026-05-04"}
  ]
}
```

- [ ] __Step 2: Verify it parses__

```bash
cd ~/claude-personal-trainer && python3 -m json.tool examples/strength-and-conditioning-recreational-athlete/data.example.json > /dev/null && echo OK
```
Expected: `OK`

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/strength-and-conditioning-recreational-athlete/data.example.json && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add example data.example.json (schema only, synthetic data)"
```

---

## Task E.3: Copy de-personalized knowledge files into the example

__Files:__
- Create: `~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/knowledge/`

- [ ] __Step 1: Copy from `knowledge-bank/`__

```bash
mkdir -p ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/knowledge && \
  cp -r ~/claude-personal-trainer/knowledge-bank/strength \
        ~/claude-personal-trainer/knowledge-bank/tendinopathy \
        ~/claude-personal-trainer/knowledge-bank/nutrition \
        ~/claude-personal-trainer/knowledge-bank/recovery \
        ~/claude-personal-trainer/knowledge-bank/body-composition \
        ~/claude-personal-trainer/knowledge-bank/general-evidence-discipline \
        ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/knowledge/
```

- [ ] __Step 2: Verify__

```bash
find ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/knowledge -type f | sort
```
Expected: ~12 files matching the bank

- [ ] __Step 3: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/strength-and-conditioning-recreational-athlete/knowledge/ && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add example knowledge/ (copied from knowledge-bank, already de-personalized)"
```

---

## Task E.4: Compose the example `index.html`

__Files:__
- Create: `~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/index.html`

- [ ] __Step 1: Use the dashboard shell + 4 widgets__

```bash
cd ~/claude-personal-trainer && python3 <<'EOF'
import pathlib
shell = pathlib.Path('site-templates/dashboard-shell.html').read_text()
widgets = {}
for w in ['streak-card', 'progress-chart', 'timeline-table', 'kpi-tiles']:
    widgets[w] = pathlib.Path(f'widgets/{w}.html').read_text()

today_block = '<section>' + widgets['kpi-tiles'] + '</section>\n<section>' + widgets['streak-card'] + '</section>'
history_block = '<section>' + widgets['progress-chart'] + '</section>\n<section>' + widgets['timeline-table'] + '</section>'

out = (shell
       .replace('{{DOMAIN_TITLE}}', 'Strength &amp; Conditioning — Example')
       .replace('{{USER_TAGLINE}}', 'In-season recreational athlete (synthetic example)')
       .replace('{{TODAY_WIDGETS}}', today_block)
       .replace('{{HISTORY_WIDGETS}}', history_block))

# Patch widgets to fetch from data.example.json since this is the example
out = out.replace("fetch('./data.json')", "fetch('./data.example.json')")

pathlib.Path('examples/strength-and-conditioning-recreational-athlete/index.html').write_text(out)
print('Wrote example index.html')
EOF
```

- [ ] __Step 2: Smoke test in browser__

```bash
cd ~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete && \
  python3 -m http.server 8765 &
SERVER_PID=$!
echo "Open http://localhost:8765/ — verify dashboard renders with synthetic example data"
echo "Kill: kill $SERVER_PID"
```

- [ ] __Step 3: Stop server and commit__

```bash
kill $SERVER_PID 2>/dev/null
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/strength-and-conditioning-recreational-athlete/index.html && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add example index.html composed from shell + 4 widgets"
```

---

## Task E.5: Write example `README.md`

__Files:__
- Create: `~/claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/README.md`

- [ ] __Step 1: Write the README__

```markdown
# Example: Strength and Conditioning — Recreational Athlete

This is an example fully-grown `claude-personal-trainer` setup, demonstrating what a complete repo looks like after a real user runs the wizard. The user profile here is __an in-season recreational athlete with achilles tendinopathy__, but the value of this example is broader: it shows the full shape of the framework — `CLAUDE.md` persona, `data.json` schema, `knowledge/` evidence syntheses, `index.html` dashboard.

## What's in here

- `CLAUDE.md` — the personalized coach instructions (with `{{PLACEHOLDER}}` tokens showing where the wizard would inject user-specific values)
- `data.example.json` — schema-only example data (synthetic entries, no real history)
- `knowledge/` — 11 evidence-synthesized modules covering: achilles HSR protocol, in-season volume management, protein distribution, recovery and sleep, body composition measurement, deload protocols, progressive overload, supplements, sodium/hydration, pre-game fueling, and a meta-module on citation discipline
- `index.html` — the dashboard, fetching from `data.example.json`

## How to read this example

If you're considering running `claude-personal-trainer` for your own domain (whether fitness or otherwise):
- Skim `CLAUDE.md` to see what a generated coach persona looks like
- Open `index.html` (run `python3 -m http.server` from this folder, then visit `http://localhost:8000/`)
- Browse `knowledge/` to see the evidence-discipline format that the wizard scaffolds for you

## What this example is NOT

- This is __not__ Menotti's actual training data. All entries are synthetic placeholders.
- You should not treat the prescriptions in `CLAUDE.md` as personalized advice — they're the example's defaults, not a real coach's recommendations.

## License

Inherits MIT from parent repo.
```

- [ ] __Step 2: Commit__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/strength-and-conditioning-recreational-athlete/README.md && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "Add example README explaining purpose + structure"
```

---

## Task E.6: PII review pass

__Files:__
- May modify any file in `examples/strength-and-conditioning-recreational-athlete/`

- [ ] __Step 1: Grep for personal identifiers__

```bash
cd ~/claude-personal-trainer && \
  grep -ri 'menotti\|menottim\|@netflix\|netflix.com' examples/ | grep -v 'placeholder\|synthetic'
```
Expected: NO matches

- [ ] __Step 2: Grep for specific dates that match real history__

```bash
grep -r '2026-04\|2026-05-0' examples/ | grep -v 'placeholder\|synthetic'
```
Acceptable: only synthetic-example dates (e.g. `2026-05-01`, `2026-05-03`) in `data.example.json`. Real dates from Menotti's actual training (Apr 22 plateau break, etc.) must NOT appear.

- [ ] __Step 3: Grep for specific real weights / lifts__

```bash
grep -r '230\|235\|240\|225 lbs\|143 lb\|148 lb' examples/ | head -20
```
Review each match. Acceptable: round-number example weights in synthetic data (`220 lbs`, `225 lbs`, `145 lbs`). Unacceptable: any specific weight from Menotti's real progression sequence.

- [ ] __Step 4: Manually skim CLAUDE.md for narrative leaks__

Look for: specific incident references, dated events, "as of week 9," "in April we tried...", references to particular knowledge gaps. These are PII even without a name. Replace with generic protocol prose.

- [ ] __Step 5: Commit any fixes__

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    add examples/ && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -m "PII review pass on bundled example: scrub residual personal references"
```

---

# PHASE F: Migration + Public Launch

## Task F.1: Make existing `training-program` repo private

__Files:__
- No code change. GitHub repo settings change.

- [ ] __Step 1: Navigate to GitHub settings__

Visit: https://github.com/menottim/training-program/settings

- [ ] __Step 2: Scroll to "Danger Zone" -> "Change repository visibility" -> set to Private__

Confirm with the typed repo name. The site at `https://menottim.github.io/training-program/` will stop serving — accept this.

- [ ] __Step 3: Verify site no longer publicly accessible__

```bash
curl -sI https://menottim.github.io/training-program/ | head -1
```
Expected: 404 or similar

---

## Task F.2: Create new private `training-program-personal` daily-driver

- [ ] __Step 1: Create empty private repo on GitHub__

Visit: https://github.com/new
- Owner: menottim
- Name: `training-program-personal`
- Visibility: Private
- Do NOT initialize with README

- [ ] __Step 2: Add new remote and push__

```bash
cd ~/training-program && \
  git remote add personal git@github.com:menottim/training-program-personal.git && \
  git push personal main
```

- [ ] __Step 3: Verify push succeeded__

```bash
curl -sI -H "Authorization: token $(gh auth token)" https://api.github.com/repos/menottim/training-program-personal | head -1
```
Expected: HTTP 200

- [ ] __Step 4: Update local remotes — rename origin to archived, personal to origin__

```bash
cd ~/training-program && \
  git remote rename origin archived && \
  git remote rename personal origin && \
  git remote -v
```
Expected: `origin` -> `training-program-personal`, `archived` -> `training-program`

---

## Task F.3: Push `claude-personal-trainer` to public GitHub

- [ ] __Step 1: Create empty public repo on GitHub__

Visit: https://github.com/new
- Owner: menottim
- Name: `claude-personal-trainer`
- Visibility: Public
- Do NOT initialize

- [ ] __Step 2: Add remote and push__

```bash
cd ~/claude-personal-trainer && \
  git remote add origin git@github.com:menottim/claude-personal-trainer.git && \
  git push -u origin main
```

- [ ] __Step 3: Verify visibility__

```bash
curl -sI https://github.com/menottim/claude-personal-trainer | head -1
```
Expected: HTTP 200

- [ ] __Step 4: Open repo in browser; verify README renders__

---

## Task F.4: End-to-end fork test

- [ ] __Step 1: Clone fresh to a different path__

```bash
cd /tmp && rm -rf cpt-fresh-fork && \
  git clone git@github.com:menottim/claude-personal-trainer.git cpt-fresh-fork && \
  cd cpt-fresh-fork
```

- [ ] __Step 2: Open in Claude Code, run wizard for a third domain__

Pick something different from D.1 and D.2 — e.g., songwriting or learning Rust.

- [ ] __Step 3: Verify wizard completes and produces usable repo__

If issues arise, fix in canonical `~/claude-personal-trainer/` and push:

```bash
cd ~/claude-personal-trainer && \
  git -c user.name="menottim" -c user.email="menottim@users.noreply.github.com" \
    commit -am "Post-fork-test fixes" && git push origin main
```

Re-clone and re-test until clean.

---

## Task F.5: Optional — announcement post

If applicable: write a short announcement on Menotti's preferred channel (X, Slack, blog post). Out of scope for this plan; left to discretion.

---

# Self-Review (post-write check)

After writing the plan, I checked it against the spec:

- [x] __Spec coverage:__ Phase A covers spec Section 4.1 + Sections 5.1-5.6 (all 6 components). Phase B covers the 5 patterns + 5 widgets + 2 site templates. Phase C covers the knowledge bank seed, including both harvested and newly-authored modules. Phase D covers wizard end-to-end testing. Phase E covers the bundled example + PII review (spec Section 5.6 + Section 10 risks). Phase F covers the migration plan (spec Section 7).
- [x] __Placeholder scan:__ No `TBD`/`TODO`/etc. in plan body. Implementation roadmap timing is estimated, not promised.
- [x] __Type / file consistency:__ Pattern IDs (`time-series-numeric`, `event-log`, `streak-counter`, `periodic-review`, `tagged-collection`), widget IDs (`streak-card`, `progress-chart`, `timeline-table`, `weekly-grid`, `kpi-tiles`), knowledge tags (`strength`, `tendinopathy`, `recovery`, `nutrition`, `body-composition`, `in-season-athlete`, `evidence-discipline`) consistent across phases.
- [x] __XSS hardening:__ Every widget that renders user-controlled strings uses an `esc()` HTML-escape helper or `textContent`. SVG-based progress-chart uses `document.createElementNS` + `textContent` for tooltip data. Static markup uses template strings safely; only user data goes through `esc()`. A smoke-test step in B.6 + B.8 verifies XSS attempts fail (literal text shown, no script execution).

Total tasks: 39 across 6 phases. Estimated 6-10 days of focused work for a single engineer; some phases parallelize per spec dependency graph.
