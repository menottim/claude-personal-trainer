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
