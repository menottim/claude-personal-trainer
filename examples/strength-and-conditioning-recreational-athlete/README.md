# Example: Strength and Conditioning — Recreational Athlete

This is a fully-grown `claude-personal-trainer` setup, demonstrating what a complete repo looks like __after__ a real user runs the wizard. The user profile here is __an in-season recreational athlete with achilles tendinopathy__, but the value of this example is broader: it shows the full shape of the framework — `CLAUDE.md` persona, `data.json` schema, `knowledge/` evidence syntheses, `index.html` dashboard.

## What's in here

- `CLAUDE.md` — the personalized coach instructions. `{{PLACEHOLDER}}` tokens (e.g., `{{ATHLETE_NAME}}`) show where the wizard injects user-specific values during real setup. In this example, those tokens are left visible deliberately so a forker can see the structure.
- `data.example.json` — schema-only example data (synthetic entries, no real history). Demonstrates the 4 patterns this athlete tracks: `bodyLog` (time-series-numeric), `activityLog` (event-log), `scienceReviews` (periodic-review), `streaks` (streak-counter).
- `knowledge/` — 12 evidence-synthesized modules covering: achilles HSR protocol, in-season volume management, deload protocols, progressive overload, protein distribution, sleep + recovery, body composition measurement, supplements, hormones, sodium/hydration, pre-game fueling, and the always-included citation-discipline meta-module. Each cites named peer-reviewed sources with PMIDs/DOIs.
- `index.html` — the dashboard, fetching from `data.example.json`. Composed from 4 widgets (kpi-tiles, streak-card, progress-chart, timeline-table) inside the dashboard-shell site template.

## How to read this example

If you're considering running `claude-personal-trainer` for your own domain (whether fitness or otherwise):

1. __Skim `CLAUDE.md`__ to see what a generated coach persona looks like — voice constraints, evidence-discipline rules, tracking conventions, workflow rules.
2. __Open `index.html` in a browser.__ From this directory, run `python3 -m http.server 8000` then visit `http://localhost:8000/`. You'll see the 4-widget dashboard rendering against the synthetic `data.example.json`.
3. __Browse `knowledge/`__ to see the evidence-discipline format the wizard scaffolds for you. Each module has: scope, key findings (with evidence tiers), mechanism / practical guidance, references, corrections log, generic "How to apply" footer.

## What this example is NOT

- This is __not__ anyone's actual training data. All `bodyLog` and `activityLog` entries are synthetic placeholders.
- You should not treat the prescriptions in `CLAUDE.md` as personalized advice — they're the example's defaults, not a real coach's recommendations for you.
- The athlete profile (40-year-old recreational athlete with achilles tendinopathy playing basketball + hockey) is __one__ point in the design space the framework supports. Your wizard run will produce something different shaped to your domain.

## How this would differ for a non-fitness domain

A language-learning forker would have:
- A different `CLAUDE.md` persona (linguistics-tutor voice instead of S&C coach)
- Different `data.json` patterns (e.g., `practiceLog`, `vocabSizeLog`, `weeklyReviews`)
- Different seed `knowledge/` modules (or possibly an empty knowledge folder if no language-learning modules exist in the bank yet)
- The same `index.html` widget composition (different patterns, same widgets)

The framework is the product. Fitness is one demonstration.

## License

Inherits MIT from parent repo.
