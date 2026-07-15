---
name: write-article
description: Turn one approved research draft into a review-ready article — clean draft first, citations woven in, quality gates, then prompt-editing. Use when the user says "write the article for <slug>" or "/write-article <article-slug>". Also invoked per-article by /run-cluster.
---

# Write one article from approved research

Input: an article folder containing an approved `fact-sheet.md` (from
`/research-cluster`) and its brief. If no approved fact sheet exists, stop and
run the research-cluster flow for this article first — writing never starts
from unapproved research.

Read `config/style-guide.md` and the brief before starting.

## Stage 1 — Clean draft (no citation clutter)

Spawn a writer agent that:
- Retrieves 3–5 tone exemplars from the `blog-archive` KB for this topic.
- Writes the article from the brief + fact sheet + research-draft angles ONLY.
  **The writer has no web access and must not introduce any factual claim
  absent from the fact sheet.**
- Writes readable prose first — no inline citations yet. Follows the style
  guide, outline, and word range; places the brief's internal-link targets
  as natural inline links.

## Stage 2 — Weave citations

A separate pass maps each factual claim in the draft to its fact-sheet row and:
- inserts the inline source link at the claim (linking style per style guide),
- builds a **References** section at the end: every cited source with title,
  publisher, date, and link — ready for the writer/reviewer to spot-check.
- Writes `meta.md`: meta title (≤60 chars), meta description (≤155 chars),
  URL slug, image alt-text suggestions.

## Stage 3 — Gates (fixed order)

- **Gate 1 — fact-check**: every claim maps to a fact-sheet row; unmapped
  claim → back to the writer (max 2 rounds, then flag).
- **Gate 2 — humanizer**: style-only pass (use the humanizer skill); claims,
  numbers, links locked.
- **Gate 3 — claim diff**: re-map claims on the humanized text; revert any
  drifted wording.

## Stage 4 — Prompt editing (the human loop)

The draft is now editable by plain-language prompts from the SEO manager or
writer ("shorten the intro", "make the tone warmer in section 2", "add the
2026 EFSA update as a callout"). For every prompt edit:
1. Apply the edit.
2. Re-run the Gate 3 claim diff automatically — an edit may never change a
   claim's meaning or drop its citation. If it would, apply the stylistic part
   and tell the user which factual part needs new approved research instead.
3. Offer `/preview-article` to see the result rendered.

## Output (in `clusters/<slug>/<article-slug>/`)

`draft.md` (with inline links + References), `meta.md`, `gate-log.md`.
Ready for Gate 4 human medical + compliance review
(`config/compliance-checklist.md`). Never call it publishable before that.
