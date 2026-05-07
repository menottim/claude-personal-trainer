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
- __Use `createElement` + `textContent`__ for all user-controlled strings. __Do not use `innerHTML` with user data__, even with an escape helper. XSS prevention is a hard requirement; safe-by-construction beats safe-by-discipline.
- Under 200 LOC

__Widget skeleton (XSS-safe DOM-construction pattern, used by all shipped widgets):__

```html
<!-- WIDGET: <widget-id> -->
<!-- Renders: <pattern-id> -->
<!-- XSS-safe: uses DOM construction (createElement + textContent), no innerHTML on user data -->
<style>
  /* widget-scoped CSS */
</style>

<h3>Title</h3>
<div id="<widget-id>-mount">Loading...</div>

<script>
(async () => {
  const data = await fetch('./data.json').then(r => r.json());
  const items = Array.isArray(data.someKey) ? data.someKey : [];
  const mount = document.getElementById('<widget-id>-mount');
  while (mount.firstChild) mount.removeChild(mount.firstChild);
  if (items.length === 0) {
    mount.textContent = 'No data yet.';
    return;
  }
  items.forEach(item => {
    const el = document.createElement('div');
    el.className = 'widget-row';
    // For ANY user-controlled string, use textContent (it auto-escapes):
    el.textContent = item.userControlledString == null ? '' : String(item.userControlledString);
    mount.appendChild(el);
  });
})();
</script>
```

For SVG widgets, use `document.createElementNS('http://www.w3.org/2000/svg', '...')` and set tooltip/label content via `textContent`. Never set `innerHTML` on an SVG element with user data.

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
