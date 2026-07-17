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
- Includes a **Table of Contents** (numbered anchor links to every H2 except
  References) placed after the intro/definition block — a house convention
  present across the corpus; never skip it.

## Stage 2 — Weave citations

A separate pass maps each factual claim in the draft to its fact-sheet row and:
- inserts the inline source link at the claim (linking style per style guide),
- builds a **References** section at the end with title, publisher, date, and
  link — ready for the writer/reviewer to spot-check.
- **Reference cap — at most 3 references per article** (manager policy,
  2026-07-17). Keep the 3 most authoritative, most load-bearing sources
  (prefer primary health authorities — WHO / AAP / NIH / EFSA / NHS / Cochrane —
  over weaker ones). First **consolidate**: where several claims can legitimately
  rest on one of the 3 kept sources, cite that source, so the cap costs as little
  coverage as possible. Only claims that still depend on a 4th+ source have their
  citation dropped from the article.
  - ⚠️ **Compliance tradeoff (read before shipping YMYL):** this cap overrides
    the "every claim carries its own citation" rule in
    `config/trusted-sources.md` *for the visible article*. Every claim still maps
    to a fully-sourced row in `fact-sheet.md` (the internal contract Gates 1 & 3
    enforce), so nothing is fabricated — but the rendered article may show medical
    claims with no inline citation. List every claim whose citation was dropped
    under an **"Uncited after 3-ref cap"** heading in `gate-log.md`, so the Gate 4
    medical reviewer verifies each against the fact sheet before publishing. For
    non-medical articles the cap is cosmetic; for YMYL articles it shifts
    citation-verification onto the human Gate 4.
- Writes `meta.md`: meta title (≤60 chars), meta description (≤155 chars),
  URL slug, image alt-text suggestions.

## Stage 2.5 — SEO pass (on-page optimization)

Optimize the cited draft against what actually ranks for the primary keyword:

- **If `SURFER_API_KEY` is set in `.env`** (`set -a; source .env; set +a`):
  create a Content Editor for the primary keyword via
  `POST https://app.surferseo.com/api/v1/content_editors` (header
  `API-KEY: $SURFER_API_KEY`), poll `GET .../content_editors/:id` until
  guidelines are ready, and apply its term list, heading-count range, and
  word-count range to the draft. Record the terms covered/missed in
  `gate-log.md`. Keep the style guide authoritative on voice — Surfer guides
  coverage, never tone.
- **Otherwise (internal scorer)**: pull the top-10 SERP for the primary
  keyword via Ahrefs `serp-overview`, fetch the top 5 ranking pages, extract
  their H2/H3 topics and recurring entities/terms, and check the draft covers
  every topic that ≥3 competitors cover (or consciously skips it — note why).
  Also set a word-count sanity range from the competitor median.
- Either way: never add a term by stuffing — coverage gaps are fixed by
  adding a real, fact-sheet-backed passage, or flagged for new research.

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
3. Re-render the preview at its existing URL.
4. **Fix the machine, not just the article.** If the edit reveals a
   systematic gap — a house convention the writer missed, a recurring style
   or structure issue, anything that would repeat on the next article —
   ALSO apply it at the source: update the style guide or the relevant
   skill (through the update-skill flow once the repo has a remote), bump
   the plugin version, and tell the manager what was made permanent. A
   correction given once should never need to be given twice.

## Output (in `clusters/<slug>/<article-slug>/`)

`draft.md` (with inline links + References), `meta.md`, `gate-log.md`.
Ready for Gate 4 human medical + compliance review
(`config/compliance-checklist.md`). Never call it publishable before that.
