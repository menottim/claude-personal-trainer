# `claude-personal-trainer` — Design Spec

__Status:__ Draft, awaiting review
__Date:__ 2026-05-07
__Author:__ Menotti Minutillo (`menottim`)
__Target repo:__ `github.com/menottim/claude-personal-trainer` (public)

---

## 1. Problem & Goal

Menotti has built `training-program/` over ~9 weeks: a static GitHub Pages site backed by `data.json`, with a `CLAUDE.md` that turns Claude Code into a CSCS-grade strength-and-conditioning coach for an in-season recreational athlete with bilateral achilles tendinopathy. The pattern is working — Claude logs activities to `data.json`, runs weekly science reviews, writes evidence-synthesis knowledge files with verified citations, and keeps the live site in sync.

The pattern is __domain-portable__. The same shape (`CLAUDE.md` persona + `data.json` + `knowledge/` + static dashboard) fits any personal practice that benefits from longitudinal tracking and an evidence-anchored coach: language learning, financial planning, songwriting, woodworking, hobby photography, etc. The current repo is hardcoded for one user with one specific profile.

__Goal:__ ship a public, forkable template — `claude-personal-trainer` — that lets anyone scaffold their own personalized version through a Claude-driven conversational setup. The forker's resulting repo should be __their own__ in voice, schema, and content; no preset assumes they want Menotti's routine.

## 2. Non-Goals (v1)

- Be a fitness app. Fitness is a __use case__; the framework is the product.
- Bundle expert-grade knowledge for every domain on Day 1. Seed enough to prove the pattern; let domain-experts contribute over time.
- Support platforms beyond Claude Code. (`CLAUDE.md` bootstrap mechanism is Claude-Code-specific.)
- Ship automated tests, CI, or build pipelines. v1 is a conversational scaffold; manual verification suffices.
- Run hosted infrastructure (no SaaS, no setup web UI, no shared deployment).
- Generate polished site UI per-domain. Ship 1-2 generic shells; users iterate via Claude later.

## 3. User Story

A new forker:

1. Visits `github.com/menottim/claude-personal-trainer`. Reads README; sees one example (the de-personalized strength-and-conditioning repo) demonstrating an end-state.
2. Forks the repo, clones locally, opens in Claude Code.
3. Says anything to Claude. Bootstrap `CLAUDE.md` detects no `data.json` exists → enters wizard mode.
4. Wizard interviews them across 5 phases (multi-turn conversational, not a rigid form): __Domain & Goals → Voice & Coaching Style → Tracking Preferences → Evidence Standards → Site / Artifact preferences__.
5. Wizard generates: personalized `CLAUDE.md` (replaces bootstrap), composed `data.json` (from patterns), seed `knowledge/` (from knowledge-bank by topic tags), composed `index.html` (from widgets + a site shell). Framework files relocate to `.claude-personal-trainer/`.
6. Wizard ends with a self-check ("here's what I generated; verify, tell me to fix anything"). User reviews, iterates with Claude, makes a first commit.
7. Day 2+: User talks to Claude as a coach in their domain. Claude logs to `data.json`, writes new knowledge files (with web verification per the user's chosen evidence standards), and keeps the static site in sync. Same daily-driver experience Menotti has today.

## 4. Architecture

### 4.1 Day-0 repo state (what someone forks)

```
claude-personal-trainer/
├── README.md                       Project overview, "open in Claude Code to start"
├── LICENSE                         MIT
├── CONTRIBUTING.md                 How to add modules / patterns / widgets / examples
├── CLAUDE.md                       BOOTSTRAP — drives the setup interview
├── wizard/                         Structured Claude-readable prompts (not executable code)
│   ├── interview.md                5-phase interview structure, prompts, branching
│   ├── generation-rules.md         How to compose patterns/widgets into user files
│   ├── claude-md-template.md       Meta-template for the user's personalized CLAUDE.md
│   └── archive-rules.md            What to archive after setup, where it goes
├── patterns/                       JSON schema fragments for data.json composition
│   ├── time-series-numeric.json    e.g. weight, mood, blood-pressure, vocab-count
│   ├── event-log.json              e.g. workouts, language sessions, songs written
│   ├── streak-counter.json         e.g. consecutive practice days, sober streak
│   ├── periodic-review.json        e.g. weekly retros, monthly check-ins, science review
│   └── tagged-collection.json      e.g. recipes, song ideas, study cards
├── widgets/                        HTML widget templates for index.html composition
│   ├── streak-card.html            Reads streak-counter pattern
│   ├── progress-chart.html         Reads time-series-numeric pattern
│   ├── timeline-table.html         Reads event-log pattern
│   ├── weekly-grid.html            Reads event-log pattern with a week schema
│   └── kpi-tiles.html              Generic key-stat tiles
├── knowledge-bank/                 Topic-tagged evidence modules (seed library)
│   ├── _index.json                 tag → files mapping for wizard lookup
│   ├── strength/
│   │   ├── progressive-overload.md
│   │   ├── deload-protocols.md
│   │   └── in-season-volume.md
│   ├── recovery/
│   │   ├── sleep-and-performance.md
│   │   └── recovery-modalities.md
│   ├── nutrition/
│   │   ├── protein-distribution.md
│   │   └── pre-game-fueling.md
│   ├── tendinopathy/
│   │   └── achilles-hsr-protocol.md
│   ├── body-composition/
│   │   └── measurement-methods.md
│   └── general-evidence-discipline/
│       └── how-to-cite.md          Meta-module: rules for verified citations
├── site-templates/                 Minimal generic site shells
│   ├── dashboard-shell.html        Tabbed: Today / History / Knowledge
│   └── log-viewer-shell.html       Single-page chronological log
└── examples/                       Fully-grown reference repos
    └── strength-and-conditioning-recreational-athlete/
        ├── CLAUDE.md               De-personalized version of Menotti's
        ├── data.example.json       Schema only, no real activity log
        ├── knowledge/              Synthesized evidence files
        ├── index.html              The polished fitness dashboard
        └── README.md               Explains what this example demonstrates
```

### 4.2 Day-N repo state (after wizard runs)

```
their-repo/
├── README.md                       Personalized: their domain, goals, getting-started
├── CLAUDE.md                       PERSONALIZED — replaces bootstrap; drives daily coaching
├── data.json                       Schema composed from patterns/, real data
├── knowledge/                      Seed modules pulled from knowledge-bank/ + grow-over-time
├── index.html                      Composed from widgets/ + a site-templates/ shell
└── .claude-personal-trainer/       Framework files moved here, available for future use
    ├── wizard/
    ├── patterns/
    ├── widgets/
    ├── knowledge-bank/
    └── examples/
```

The framework files persist in `.claude-personal-trainer/` so the user can later add tracking dimensions, pull more knowledge modules, or re-skin the site without re-cloning upstream.

## 5. Components

### 5.1 Bootstrap `CLAUDE.md`

The Day-0 `CLAUDE.md`. The wizard is __not executable code__ — it's structured prompts in `wizard/*.md` that Claude reads and follows. The bootstrap `CLAUDE.md` orchestrates the whole flow.

Its job:
- On first conversation, detect the absence of `data.json` (or presence of a `.wizard-state.json` indicating a paused setup) → enter wizard mode.
- Read `wizard/interview.md` and conduct the 5-phase interview.
- Read `wizard/generation-rules.md` and assemble outputs.
- Once setup completes, __overwrite itself__ with the personalized CLAUDE.md and move framework dirs into `.claude-personal-trainer/`.
- Hand off to the user with a clear "setup done, here's how to use this" message.

### 5.2 Patterns library

Reusable JSON shapes the wizard composes into the user's `data.json`. Each pattern is a JSON Schema fragment + an example block + a brief docstring. Examples:

__`time-series-numeric.json`__ — for tracking a numeric over time (weight, mood, lines-of-poetry-written, vocabulary-count). Shape:

```json
{
  "id": "time-series-numeric",
  "schema": {
    "type": "array",
    "items": {
      "type": "object",
      "required": ["date", "value"],
      "properties": {
        "date": {"type": "string", "format": "date"},
        "value": {"type": "number"},
        "unit": {"type": "string"},
        "notes": {"type": "string"}
      }
    }
  },
  "exampleBlock": [
    {"date": "2026-05-07", "value": 220.5, "unit": "lbs", "notes": "AM weigh-in"}
  ],
  "rendererHint": "progress-chart"
}
```

__`event-log.json`__ — for tracking discrete events with type/intensity/duration (workouts, practice sessions, journal entries).

__`streak-counter.json`__ — for habits with streak semantics (consecutive practice days, sober streak, no-injury days).

__`periodic-review.json`__ — for retros (weekly / monthly / quarterly check-ins).

__`tagged-collection.json`__ — for collections of items with tags (recipes, song ideas, study cards, project notes).

The wizard picks 1-N patterns based on what the user said they want to track. Pattern composition: each pattern becomes a top-level array key in the user's `data.json`. e.g., a fitness user might end up with `data.json` keyed `bodyLog`, `activityLog`, `scienceReviews`. A language learner might have `practiceLog`, `vocabSize`, `weeklyReviews`.

### 5.3 Widgets library

Reusable HTML+JS components rendered into `index.html`. Each widget reads a specific pattern's schema and renders. Mapping is roughly 1:1 — a `streak-counter` pattern in `data.json` gets a `streak-card.html` widget on the dashboard. A `time-series-numeric` pattern gets a `progress-chart.html` widget.

Widgets are __vanilla HTML/CSS/JS__ — no framework dependency. They read from `data.json` via a single fetch, transform, and render. This matches the existing `training-program/index.html` pattern. Each widget is < 200 LOC, self-contained.

### 5.4 Knowledge bank

Topic-tagged markdown evidence modules. v1 seeds with ~10 modules harvested + de-personalized from Menotti's existing `knowledge/` folder. Tags follow a controlled vocabulary in `_index.json`:

```json
{
  "modules": {
    "strength/progressive-overload.md": ["strength", "intermediate-lifter", "double-progression"],
    "recovery/sleep-and-performance.md": ["sleep", "athletic-performance", "cognition"],
    "tendinopathy/achilles-hsr-protocol.md": ["achilles", "tendinopathy", "hsr", "beyer-2015"]
  },
  "tagDocs": {
    "strength": "Resistance training, hypertrophy, strength adaptations",
    "tendinopathy": "Tendon injuries, loading protocols, return-to-play"
  }
}
```

Wizard reads user interview answers, runs a tag-match against `_index.json`, copies relevant modules into the user's `knowledge/`. No bundled knowledge for unsupported domains; those forkers start with an empty `knowledge/` and grow it over time via web-search-on-demand during normal coaching.

Modules use the same evidence-discipline format as the existing fitness ones: __named citations__ (author, year, journal, PMID/DOI), __evidence tiers__ (Strong / Moderate / Emerging), __corrections logs__ for revisions.

### 5.5 Site templates

1-2 minimal HTML shells widgets compose into. Generic styling (system fonts, neutral palette). Day-N user can ask Claude to re-skin without touching the framework.

- `dashboard-shell.html` — tabbed layout (Today / History / Knowledge), shows widgets in a grid.
- `log-viewer-shell.html` — single-page chronological log, simpler.

### 5.6 Examples

Fully-grown reference repos. v1 ships __one__: a de-personalized version of Menotti's strength-and-conditioning repo. Demonstrates what a complete setup looks like for the most-likely-forker (another in-season athlete). Community PRs add more examples over time.

The example __must not__ contain real personal data (no real activityLog entries, no real bodyLog stats). It contains: the polished CLAUDE.md (with athlete profile placeholders), a `data.example.json` showing schema shape only, the synthesized knowledge files (the actual research), and a working `index.html`.

## 6. The Wizard Interview

### 6.1 Five phases

Each phase is multi-turn natural conversation, not a fixed-question form. Wizard reads `wizard/interview.md` for prompts and branching guidance.

__Phase 1 — Domain & Goals (5-10 min):__
- What domain / practice / goal does this repo support?
- What does success look like in 6 / 12 months?
- Who are you (relevant context: profession, time available, constraints, prior experience)?
- Any injuries, conditions, or limitations that should shape advice?

__Phase 2 — Voice & Coaching Style (3-5 min):__
- Direct vs. warm vs. analytical vs. playful?
- Should Claude push back when you're wrong, or stay supportive?
- Any AI-writing patterns you hate (em-dashes, "let's dive in," motivational fluff)?
- Pronoun preferences, name spellings, addressing style?

__Phase 3 — Tracking Preferences (5-10 min):__
- What do you want to log? (open-ended, then mapped to patterns)
- How often — daily, after each session, weekly retro?
- Detail level — terse summaries or full transcripts?
- Body / output stats: which matter, which don't?
- Streaks vs. cumulative totals vs. rolling averages?

__Phase 4 — Evidence Standards (3-5 min):__
- How rigorous on citations? Strict peer-review (PubMed-grade) or "Claude's best understanding"?
- Should Claude refuse to assert things without sources, or speculate-with-flag?
- Want a `knowledge/` folder that grows over time, or just live data?

__Phase 5 — Site / Artifact Preferences (3-5 min):__
- Do you want a visual dashboard or just data + Claude conversations?
- Public artifact (GitHub Pages) or private/local?
- Privacy default: this resulting repo is __strongly recommended private__ for any health/financial/personal data; explicit opt-in to make public.

Total: ~20-35 min for a thorough setup. Wizard saves state after each phase to `.wizard-state.json` so user can pause/resume.

### 6.2 Generation step

After phase 5, wizard:
1. Composes `data.json` schema from selected patterns + writes example data
2. Reads tags from interview answers, copies matching knowledge modules to `knowledge/`
3. Generates personalized `CLAUDE.md` from `wizard/claude-md-template.md` populated with:
   - Domain & goals (Phase 1)
   - Coaching voice (Phase 2)
   - Tracking conventions (Phase 3)
   - Evidence-discipline rules (Phase 4)
   - Workflow rules (when to log, when to commit, when to write knowledge files)
4. Composes `index.html` from widgets that match the user's chosen patterns + a site shell
5. Writes a personalized `README.md` (their domain, getting-started, "delete data.json to re-run setup")
6. Moves framework dirs to `.claude-personal-trainer/`
7. Self-checks: validates `data.json` parses, `index.html` opens cleanly, `CLAUDE.md` is well-formed
8. Commits the whole thing as a single "initial setup" commit using the user's git identity (asks for it if unset)

## 7. Migration Plan: existing `training-program/` repo

Three-step migration:

__Step 1: Make the existing `training-program` repo private (no rename).__ This breaks `https://menottim.github.io/training-program/` — accept this; the site has been an internal reference and the audience has been Menotti himself. Doing this without renaming preserves the URL slug in commit history and avoids GitHub squatting. Rationale: protects Menotti's real activity log + body comp data.

__Step 2: Create a new private `training-program-personal` repo as the daily driver.__ Migrate Menotti's data + CLAUDE.md + knowledge/ here; this becomes the new canonical for ongoing coaching. The original `training-program` repo becomes a graveyard (still cloneable, no longer updated).

__Step 3: De-personalize and ship as the bundled example.__ Strip Menotti's specific data (activityLog, bodyLog with real stats, real Apr-May entries). Replace with: schema-only `data.example.json`; CLAUDE.md with placeholders for athlete profile (`{{ATHLETE_NAME}}`, `{{ATHLETE_AGE}}`, `{{ATHLETE_INJURIES}}`); preserved synthesized knowledge files (the actual research is the value); a working `index.html` showing the schema rendered with example data. Ship this as `claude-personal-trainer/examples/strength-and-conditioning-recreational-athlete/`.

## 8. Implementation Roadmap

Phased build order (each phase ends in something usable):

__Phase A — Skeleton (1-2 days):__
- Initialize `claude-personal-trainer` repo with README, LICENSE, CONTRIBUTING
- Write Bootstrap `CLAUDE.md`
- Write `wizard/interview.md` with the 5-phase script
- Write `wizard/generation-rules.md`
- Write `wizard/archive-rules.md`
- Verify: someone can fork, open in Claude Code, and have a wizard conversation that successfully __understands their answers__ even before generation works

__Phase B — Patterns + Widgets (1-2 days):__
- Author the 5 patterns (`time-series-numeric`, `event-log`, `streak-counter`, `periodic-review`, `tagged-collection`) with schemas + examples + renderer hints
- Author the 5 widgets matching them
- Author 1-2 site templates (`dashboard-shell`, `log-viewer-shell`)
- Verify: hand-composed test outputs (one per pattern) render correctly in browser

__Phase C — Knowledge Bank Seed (1 day):__
- Harvest + de-personalize ~10 modules from Menotti's `training-program/knowledge/`
- Re-tag against the controlled vocabulary in `_index.json`
- Write `general-evidence-discipline/how-to-cite.md` (the meta-module on citation discipline)
- Verify: a tag-match against synthetic interview answers returns sensible modules

__Phase D — Wizard Generation (2-3 days):__
- Implement (in `wizard/generation-rules.md` as Claude-readable rules) the composition logic: interview answers → data.json schema, → knowledge/ files, → index.html, → personalized CLAUDE.md
- End-to-end test: Menotti runs the wizard for a fictional non-fitness domain (e.g., language learning); verify outputs are coherent and usable
- End-to-end test 2: Menotti runs the wizard for a strength-and-conditioning profile and verifies it produces something equivalent to a fresh version of his own setup

__Phase E — Bundled Example (1 day):__
- De-personalize current `training-program/` per migration step 3
- Drop into `examples/strength-and-conditioning-recreational-athlete/`
- Verify: the example renders correctly with example data; a forker reading the example can understand the pattern

__Phase F — Migration + Public Launch (1 day):__
- Execute migration steps 1-2 for Menotti's personal repo
- Push `claude-personal-trainer` to public GitHub
- Update `claude-personal-trainer` README with links to the example, contributor guide, and "open in Claude Code to start"
- Optional: Slack / X / wherever Menotti announces this

Total estimated effort: __6-10 days of focused work__.

__Dependency graph__: Phase A (skeleton + wizard prompts), B (patterns + widgets), and C (knowledge bank seed) are independent and can run in parallel. Phase D (wizard generation logic) depends on A + B + C being done. Phase E (bundled example) can run in parallel with B/C/D once A is settled (it just needs the architecture decided). Phase F (migration + public launch) is final and depends on all prior.

## 9. Decisions & Tradeoffs

- __Bootstrap `CLAUDE.md` mechanism vs. CLI scaffolder vs. skill__ — Chose bootstrap-CLAUDE.md (option 1 from brainstorm Q2). Lowest infrastructure, leverages Claude's interview-and-adapt strength, ties the user into Claude Code from minute one (which is where they'll be doing day-to-day coaching anyway).
- __Hybrid knowledge sourcing__ (seed bank + grow-over-time) — Chose this (option D from brainstorm Q3) over pre-curated-only or web-search-only. Day-1 forkers in covered domains get immediate value; uncovered domains aren't blocked; web-search-on-demand fills gaps during real coaching.
- __Pattern + widget library__ vs. wizard-generates-from-scratch — Chose patterns/widgets (Approach 2 from Q4). Reusable artifacts encode known-good schemas; wizard intelligence stays focused on composition + voice + content; community can PR new patterns/widgets.
- __Framework files survive into `.claude-personal-trainer/`__ vs. delete-after-setup — Chose survive. Lets users add tracking dimensions or pull knowledge modules later without re-cloning. Hidden enough not to clutter top-level.
- __Ship one example, not many__ (Approach 1 from Q4 → revised to "no preset, framework-first") — v1 ships the strength-and-conditioning example (because that's what exists) but no preset is wired into the wizard. The example is reference, not default. Other domains added via community contribution.
- __Privacy default: private + opt-in to public__ — Wizard recommends private repo for any health/finance/personal data. Public path is supported but explicit. Avoids the "user pushes their bodyfat percentage to a public repo without thinking about it" failure mode.
- __Site default: local-only__ — `index.html` opens in browser locally; GitHub Pages enabling is optional opt-in step. Sidesteps the "private repo + Pages requires Pro" tension.
- __Platform: Claude Code only for v1__ — Bootstrap-CLAUDE.md is Claude-Code-specific. Other LLM-tool variants (Cursor, Aider, Claude Desktop) deferred to v2 if there's demand.

## 10. Open Questions / Risks

- __Wizard quality is conversational__ — there's no automated test that "the wizard does a good interview." First few real users (Menotti + 1-2 trusted others) drive iteration. Acceptable for v1 given the scope.
- __Citation hallucination risk during grow-over-time knowledge writes__ — Mitigated by personalized CLAUDE.md inheriting strict evidence-discipline rules from `general-evidence-discipline/how-to-cite.md`. Worth re-reviewing after first few real-world knowledge files are generated.
- __Pattern composition could produce semantically odd `data.json`__ (e.g., user picks 4 patterns that don't naturally cohere). Wizard asks clarifying questions to avoid; if it slips through, user can ask Claude to refactor.
- __Example de-personalization risk__ — Menotti must verify no real PII slips into the bundled example before public push. One round of careful review at Phase E.
- __Adoption / discoverability__ — Out of scope for spec; v1 ships, success is measured by whether a non-Menotti human successfully sets up and uses it.

## 11. Self-Review Checklist

- [x] Placeholder scan — no TBDs / TODOs in the spec body. Implementation roadmap timing is estimated, not promised.
- [x] Internal consistency — architecture (Section 4) matches components (Section 5) matches wizard generation (Section 6.2).
- [x] Scope check — single coherent v1; phased implementation roadmap fits in one writing-plans cycle.
- [x] Ambiguity check — controlled vocabulary for tags is named (`_index.json`), pattern naming convention is fixed, file paths are specified, the migration plan is explicit on which slug becomes private vs. graveyard.

---

__Next step:__ user reviews this spec, requests changes if needed, then implementation plan via `superpowers:writing-plans` skill.
