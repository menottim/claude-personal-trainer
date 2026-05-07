# claude-personal-trainer

A forkable [Claude Code](https://claude.com/claude-code) template that scaffolds a personalized longitudinal-tracking-and-coaching repo for any practice you care about: fitness, language learning, financial planning, songwriting, woodworking, anything where you want Claude to act as a coach over months and years rather than one chat at a time.

## Why this exists

If you've already used Claude as a coach for something, you've probably noticed the limits:

- Your data lives in chat scrollback. You can't graph it, share it, or trust that next session's Claude remembers what last session's Claude saw.
- Claude's recommendations drift over time. Without a knowledge base anchored to specific sources, you end up re-litigating the same questions every few weeks.
- Claude's voice resets every conversation. The persona you spent 20 minutes calibrating in March is gone by April.
- A glanceable dashboard is genuinely useful for things you track for a year-plus, but nobody wants to build one from scratch.

This repo solves that. You fork it, run a 30-minute setup interview with Claude, and end up with a personalized repo that handles all four problems above for your specific practice.

## How it works

```
1. Fork this repo on GitHub
2. git clone git@github.com:<you>/claude-personal-trainer.git
3. cd claude-personal-trainer && claude
4. Say anything to Claude.
```

The `CLAUDE.md` at the root is a __bootstrap__ that detects no `data.json` exists and runs an interactive 5-phase interview:

1. __Domain and goals.__ What practice are you tracking? What does success look like in 6 / 12 months? Any constraints or sensitivities that should shape advice?
2. __Voice and coaching style.__ How should Claude talk to you? What AI-writing patterns do you hate?
3. __Tracking preferences.__ What do you actually want to log, and how often?
4. __Evidence standards.__ How rigorous on citations? Strict peer-review, moderate, or just Claude's best understanding?
5. __Site and artifact preferences.__ Visual dashboard or just data? Public or private?

When the interview finishes, the wizard generates:

- A personalized `CLAUDE.md` with the persona, voice constraints, evidence rules, and tracking conventions you specified
- A `data.json` shaped to your tracking patterns
- A seed `knowledge/` folder with relevant evidence modules pulled from the bundled topic-tagged bank (or empty if your domain isn't covered yet, ready to grow as topics come up)
- An `index.html` dashboard composed from widgets that match your patterns (optional)

After that, you talk to Claude as your coach. It logs sessions, writes new knowledge files with verified citations when topics come up, runs periodic reviews, and keeps the dashboard in sync. Same daily-driver experience the maintainer has been using on his own training repo for ~9 weeks.

## See a finished one

`examples/strength-and-conditioning-recreational-athlete/` is a worked example showing what a complete setup looks like once the wizard has run. The placeholder tokens (`{{ATHLETE_NAME}}`, `{{ATHLETE_AGE}}`, etc.) are deliberately left visible so you can see where the wizard would inject your specific values during real setup.

That example is shaped for an in-season recreational athlete with achilles tendinopathy. Different domains will look different in `CLAUDE.md` and `data.json` but share the same overall shape.

## Privacy

The wizard recommends keeping your resulting repo private by default if you're tracking health, financial, or personal data. Public is fully supported, but the wizard walks you through a checklist first: `data.json` is world-readable in a public repo, your git commit email is published with every commit, you can't easily go private later, and so on.

GitHub Pages on a private repo requires a paid GitHub plan. On the free tier, choosing "private" defaults the dashboard to local-only, which is fine for most users.

## Architecture

```
CLAUDE.md             Bootstrap. Replaced with your personalized version after setup.
wizard/               Interview script, generation rules, meta-templates, archive rules.
patterns/             5 reusable JSON schema fragments composed into your data.json.
widgets/              5 vanilla HTML+JS components for your dashboard. XSS-safe by construction.
site-templates/       2 generic site shells (tabbed dashboard + single-page log).
knowledge-bank/       10 topic-tagged seed evidence modules.
examples/             Worked-example repos showing finished setups.
```

After setup, the framework directories relocate to `.claude-personal-trainer/` so your repo top-level shows only your personalized files.

## Status

v1. Tested manually for the strength-and-conditioning case (the maintainer's actual coaching loop) and walked through a fictional language-learner persona to harden the wizard prompts for non-fitness domains. Not battle-tested by external users yet. If you fork it and something breaks, file an issue.

## Contributing

See `CONTRIBUTING.md` for how to add patterns, widgets, knowledge modules, or worked examples.

## License

MIT (see `LICENSE`)
