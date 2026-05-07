# claude-personal-trainer

A forkable [Claude Code](https://claude.com/claude-code) template that turns Claude into your personal strength-and-conditioning coach: tracks your training and body composition over months, maintains an evidence-anchored knowledge base of real research (achilles HSR, in-season volume, deload protocols, protein distribution, recovery, body comp, fueling), and gives you a static dashboard to glance at instead of scrolling through chat history.

You fork it, run a 30-minute setup interview with Claude, and end up with a personalized coaching repo for your own profile (your sport, your injury history, your strength baseline, your goals, your voice preferences). The maintainer has been using exactly this shape on his own training repo for ~9 weeks and shipped this template by generalizing it.

## Why this exists

If you've used Claude as a fitness coach, you've probably hit the limits:

- Your training data lives in chat scrollback. You can't graph it, share it, or trust that next session's Claude remembers what last session's Claude saw.
- Claude's recommendations drift over time. Without a knowledge base anchored to specific PMIDs, you re-litigate the same questions every few weeks (is creatine cycling needed? is the anabolic window 30 minutes?).
- Claude's coaching voice resets every conversation. The CSCS-grade persona you spent 20 minutes calibrating is gone by next week.
- Programming, deload triggers, and progression rules need to be __consistent__ to work, but consistency is exactly what stateless chat fails at.
- A glanceable dashboard with your lifts, body comp trends, achilles pain streaks, and weekly volume against Schoenfeld 2017 minimums is genuinely useful. Nobody wants to build one from scratch.

This repo solves all five.

## What you get out of the box

- A bootstrap `CLAUDE.md` that interviews you across 5 phases (your sport + injuries + goals, your voice preferences, what you want to log, evidence-discipline level, dashboard preferences).
- A `data.json` shaped for the things fitness people actually track: workouts, body composition, sleep, achilles or other injury pain monitoring, periodic science reviews, habit streaks.
- A seed `knowledge/` folder with 10 evidence-synthesized modules: achilles HSR protocol (Beyer 2015), in-season volume management (Schoenfeld 2017, Coleman 2024), deload protocols (reactive not scheduled), progressive overload, protein distribution (Morton 2018), sleep and performance (Walsh 2021 IOC consensus), body composition measurement, sodium and hydration, pre-game fueling, plus a meta-module on citation discipline. Each cites real PMIDs/DOIs.
- An `index.html` dashboard with widgets for progress charts, streak cards, weekly schedule grid, KPI tiles, and a timeline table. Vanilla HTML+JS, XSS-safe by construction.
- A worked example at `examples/strength-and-conditioning-recreational-athlete/` showing what a complete setup looks like for an in-season recreational athlete with achilles tendinopathy.

## How to use it

```
1. Fork this repo on GitHub
2. git clone git@github.com:<you>/claude-personal-trainer.git
3. cd claude-personal-trainer && claude
4. Say anything to Claude.
```

The bootstrap detects no `data.json` exists and starts the interview. ~30 minutes later, you have a personalized repo. After that, you talk to Claude as your coach. It logs sessions, runs weekly science reviews, writes new knowledge files (with web verification) when topics come up, and keeps the dashboard in sync.

## See a finished one

`examples/strength-and-conditioning-recreational-athlete/` is the worked example. Open `index.html` from that folder in a browser to see the dashboard. Read `CLAUDE.md` to see what the wizard generates for an in-season athlete profile. The placeholder tokens (`{{ATHLETE_NAME}}`, `{{ATHLETE_AGE}}`, etc.) are deliberately visible so you can see where the wizard would inject your specific values during real setup.

## Privacy

The wizard recommends keeping your repo private if you're tracking body composition, injury status, or anything else you wouldn't want indexed by search engines. Public is supported, but the wizard walks you through a checklist first: `data.json` is world-readable in a public repo, your git commit email is published with every commit, and you can't easily go private later. GitHub Pages on a private repo requires a paid GitHub plan; without it, your dashboard is local-only (which is fine for most users; you open `index.html` from your filesystem).

## Use it for non-fitness practices?

The framework underneath is domain-agnostic. The patterns (time-series, event-log, streak-counter, periodic-review, tagged-collection), widgets, and citation-discipline meta-module are reusable. If you fork this for language learning, financial tracking, songwriting, or some other longitudinal practice, the wizard handles your domain. You'll just have an empty `knowledge/` folder to start, since v1 only ships fitness modules. Claude will research and synthesize new knowledge files as topics come up in your coaching, with the same evidence-discipline rules applied.

That's a v2 path, not the v1 promise. v1 is a fitness coach.

## Architecture

```
CLAUDE.md             Bootstrap. Replaced with your personalized version after setup.
wizard/               Interview script, generation rules, meta-templates, archive rules.
patterns/             5 reusable JSON schema fragments composed into your data.json.
widgets/              5 vanilla HTML+JS components for your dashboard. XSS-safe by construction.
site-templates/       2 generic site shells (tabbed dashboard + single-page log).
knowledge-bank/       10 topic-tagged seed evidence modules (fitness/strength/recovery/nutrition).
examples/             Worked-example repos showing finished setups.
```

After setup, the framework directories relocate to `.claude-personal-trainer/` so your repo top-level shows only your personalized files.

## Status

v1. Tested on the maintainer's own coaching loop for ~9 weeks before this generalization, plus a fictional language-learner persona walkthrough to harden the wizard for non-fitness forks. Not battle-tested by external users yet. If you fork it and something breaks, file an issue.

## Contributing

See `CONTRIBUTING.md` for how to add patterns, widgets, knowledge modules, or worked examples. New knowledge modules need verified peer-reviewed citations (no fabricated references); examples need a PII review before merge.

## License

MIT (see `LICENSE`)
