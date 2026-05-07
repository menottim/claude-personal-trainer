# Wizard Interview Script

Conduct each phase as multi-turn natural conversation. Do not list the prompts as a form. Follow up on interesting answers; skip prompts that don't apply. Save state to `.wizard-state.json` after each phase.

## Phase 1 — Domain and Goals (5-10 min)

Goal: understand what the user is trying to do, who they are, and what success looks like.

Open with: "Let's start with what you're trying to do. In a sentence or two, what's the practice or goal this repo is going to support?"

Then explore:
- What does success look like in 6 / 12 months?
- Who are you in this context: profession, time available per week, prior experience, constraints?
- Are there any constraints, conditions, or sensitivities that should shape advice (injuries for fitness, dietary restrictions for nutrition, learning differences for study practices, time-of-day limits, etc.)?
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
- Output metrics: which matter, which don't? (e.g., body comp for fitness, vocab count or words-written for language, hours-deep-work for focus, dollars saved for finance)
- Confirm: list the patterns you'd recommend and ask for approval / additions / removals

State to save: `tracking_patterns` (array of pattern IDs to compose), `pattern_field_overrides` (any user customizations).

## Phase 4 — Evidence Standards (3-5 min)

Goal: how rigorous on citations? Set the discipline level for knowledge files.

Open with: "How rigorous do you want me to be on sources? Some users want strict peer-review citations with PMID/DOI. Others want me to share my best understanding without forcing a citation hunt."

Levels:
- __Strict__: every claim needs a verified primary source (peer-reviewed paper for science, governing standard for code/regulation, primary data for finance, etc.). No citing without verifying the source exists and matches the claim. Same discipline as the bundled strength-and-conditioning example.
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
- Site shell preference: tabbed dashboard or single-page log viewer?
- Widget priorities (which patterns deserve hero placement?)

### Privacy gate (mandatory before finalizing site_visibility)

Walk through this checklist explicitly with the user before they commit to a public repo. If they say "public" without considering these, run through them once and let them reconsider:

1. __`data.json` will be world-readable in a public repo.__ The dashboard URL isn't a secret; even if you don't share the URL, anyone who finds the GitHub repo can browse to `data.json` and read everything in it.
2. __Sensitive categories that often shouldn't be public__: mental-health logs, eating-disorder recovery, sobriety streaks, financial details (account numbers, balances), location traces (run routes near home), data about minor children, medical conditions, hormone panels, bodyweight if the user is sensitive about it.
3. __GitHub Pages on a private repo requires a paid GitHub plan__ (Pro/Team). On the free tier, private repos cannot host Pages. So choosing "private" defaults the dashboard to local-only — which is fine for most users.
4. __Your git commit email__ is recorded with every commit. If your `git config user.email` is set to your real email and the repo is public, every commit publishes that email. Suggest using the GitHub-supplied `<username>@users.noreply.github.com` form if they care about this. Ask the user whether they want this configured for the repo.
5. __You can always start private and go public later__ but not the reverse without forcibly rewriting history. Default to private when uncertain.

If after this walkthrough the user still chooses public, accept it and move on. Note in `state.repo_privacy` both the choice AND a brief reason ("user explicitly accepts public exposure of training data").

State to save: `site_enabled`, `site_visibility`, `site_shell`, `repo_privacy`, `noreply_email_configured`.

## End of interview

After Phase 5, summarize all 5 phases concisely back to the user and ask: "Does this look right? I'm about to generate your repo from these answers." On confirmation, proceed to `wizard/generation-rules.md`.
