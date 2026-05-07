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
   - __Phase 1: Domain and Goals__ — what are you doing, what does success look like?
   - __Phase 2: Voice and Coaching Style__ — how should Claude talk to you?
   - __Phase 3: Tracking Preferences__ — what do you want to log?
   - __Phase 4: Evidence Standards__ — citation discipline level
   - __Phase 5: Site / Artifact Preferences__ — dashboard? public or private?
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
