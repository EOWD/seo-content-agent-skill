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
  as natural inline links, following these rules:
  - **The first link in the article body must be internal** — never send a
    reader offsite before they've gone deeper into the site.
  - **One link per target page.** Never link to the same target page more
    than once in an article — one unique text link to each target is
    sufficient. Later mentions of that page stay plain text.
  - **Only relevant, natural links.** No unnecessary or repetitive internal
    links: every link is contextually relevant and reads naturally in its
    sentence. If a sentence has to be built around a link, leave it out.
  - **Collection page over individual product pages.** When a collection
    page exists for the keyword/topic being discussed, link to the
    collection, not to the individual products in it (e.g. link the HiPP
    Stage 1 collection, not HiPP UK Stage 1 and HiPP Dutch Stage 1
    separately). Confirm the collection in the Shopify catalog — never guess
    a handle.
  - **Stage/type articles also link the individual products.** When the
    article specifically discusses a formula stage or formula type (e.g.
    Stage 1 formulas), link to all relevant, currently-carried individual
    product pages for that stage/type where they fit naturally — alongside
    the collection link, one link each, no repeats.
  - **Anchor text varies** — sometimes the full target-article title,
    sometimes a short keyword phrase; never generic "click here"/"read more."
  - **Internal links are relative** (e.g. `/blogs/organicsbestclub/<slug>`,
    `/collections/<handle>`, `/products/<handle>`), never the full
    `organicsbestshop.com` domain.
  - **All links, internal and external, are clean** — no tracking or query
    parameters.
- Includes a **Table of Contents** (numbered anchor links to every H2 except
  References) placed after the intro/definition block — a house convention
  present across the corpus; never skip it.
- **Marks a visual placeholder for every visual idea in the brief** (comparison
  table, infographic) at its outline position, in the form
  `[VISUAL: <concept> — alt: "<keyword>"]`. Use the brief's specified alt-text
  keyword; if the brief didn't give one, pick the most relevant unused
  primary/secondary keyword from the fact sheet's keyword list. Never drop a
  visual concept the brief called for — placeholder it even if it isn't built
  yet. **Max 3 visuals per article** (the brief is already capped at 3 —
  never add extra ones beyond what it lists).
- **Marks a CTA placeholder under each H2 or product mention that has a real,
  currently-carried product or collection behind it**, in the form
  `[CTA: Shop <Collection/Product Name> →](/collections/<handle>)` or
  `/products/<handle>`. **The CTA links to the relevant collection page
  whenever an appropriate collection exists**; an individual product page is
  the CTA target only when no collection covers it. Pull the handle from the
  Shopify catalog — never invent a CTA target that isn't a real,
  currently-carried collection/product. Skip the CTA where no relevant
  collection or product exists rather than forcing one. The one-link-per-
  target rule applies to CTAs too: one CTA per target page per article.
- **FAQ items use H3** for each question (the "FAQ" section header itself is
  the H2) — never demote an FAQ question below H3 or promote it to H2.
- **Every heading earns its place.** Do not create H2s or H3s that are
  irrelevant, redundant, or forced in solely to include a keyword. Every
  heading must have a clear purpose and must logically introduce the content
  that follows it. Do not use "Is Stage 1 Formula Organic?" as an H2 unless
  the article specifically addresses whether Stage 1 formula is organic; do
  not place "Goat Milk Stage 1" as an H3 under "Stage 1 Formula Brands"
  unless that section specifically discusses goat milk Stage 1 brands.
  Related keywords that don't represent distinct, useful sections go into
  the prose of the section they belong to — not into separate headings.
  Follow the brief's outline, but if an outlined heading fails this test,
  fold it into the nearest relevant section and note the change in
  `gate-log.md`.
- **Closes with the standard disclaimer, verbatim.** The last block of the
  article body (after the conclusion/FAQ, before References) is the
  two-paragraph "Standard closing footer" from `config/style-guide.md`
  ("Please be aware that this information is based on general trends in
  babies…" + "Breastfeeding is the best nutrition for your baby…"), copied
  exactly — same two paragraphs, same wording and punctuation, no heading,
  not listed in the Table of Contents. Never paraphrase, shorten, merge, or
  restyle it, and never let an in-body hedge stand in for it. It is fixed
  boilerplate: the humanizer pass and prompt edits leave it untouched.

## Stage 2 — Weave citations

A separate pass maps each factual claim in the draft to its fact-sheet row and:
- inserts the inline source link at the claim (linking style per style guide).
  External reference links open in a new tab (`target="_blank" rel="noopener"`
  in the rendered HTML/preview) so the reader never leaves the page outright.
- builds a **References** section at the end with title, publisher, date, and
  link — ready for the writer/reviewer to spot-check.
- **Reference cap — 2–3 references per article, never more than 3** (manager
  policy, 2026-07-17). Keep the most authoritative, most load-bearing sources
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
- Writes `meta.md`: meta title (**must not exceed 600px** as Google renders
  it — ≤70 characters is the working proxy, go shorter when the title uses
  many wide characters — must include the primary keyword, and **never
  append "| Organic's Best" or any brand suffix automatically**; add one only
  when the manager asks for it on that article), meta description (**must
  not exceed 960px** — ≤155 characters is the working proxy — must include
  the primary keyword and end with a CTA per the style guide's pattern), a
  clean URL slug (lowercase, hyphenated, no stop-word bloat, no query
  parameters, kept well short of the 200+-character danger zone — short and
  keyword-relevant, not a restatement of the whole title), and image/visual
  alt-text suggestions (each naturally incorporating the article's primary or
  a secondary keyword — never stuffed).
- Writes `inbound-links.md` — **existing content opportunities**: existing
  articles (found via the `blog-archive` KB) that could link *to* this new
  article, prioritized by topical relevance and by where an added internal
  link would read naturally. For each: the old post's URL, the section or
  passage where the link belongs, and suggested anchor text. Suggestions
  only — old posts are edited manually at publish.

## Stage 2.5 — SEO pass (on-page optimization)

Optimize the cited draft against what actually ranks for the primary keyword:

- **If `SURFER_API_KEY` is set in `.env`** (`set -a; source .env; set +a`):
  create a Content Editor for the primary keyword via
  `POST https://app.surferseo.com/api/v1/content_editors` (header
  `API-KEY: $SURFER_API_KEY`), poll `GET .../content_editors/:id` until
  guidelines are ready, and apply its term list, heading-count range, and
  word-count range to the draft. Pull Surfer's FAQ/People-Also-Ask
  suggestions if the plan tier exposes them, and fold the relevant ones into
  an FAQ section per the style guide's ~31%-of-articles guidance. Record the
  terms covered/missed in `gate-log.md`. Keep the style guide authoritative
  on voice — Surfer guides coverage, never tone.
- **Otherwise (internal scorer)**: pull the top-10 SERP for the primary
  keyword via Ahrefs `serp-overview`, fetch the top 5 ranking pages, extract
  their H2/H3 topics and recurring entities/terms, and check the draft covers
  every topic that ≥3 competitors cover (or consciously skips it — note why).
  Also pull the "People Also Ask" questions from the SERP result and fold the
  relevant ones into an FAQ section per the same style guide guidance. Also
  set a word-count sanity range from the competitor median.
- Either way: never add a term by stuffing — coverage gaps are fixed by
  adding a real, fact-sheet-backed passage, or flagged for new research.
  Heading-count ranges and competitor headings never justify a forced
  heading: a covered topic gets its own H2/H3 only when it is a distinct,
  useful section (Stage 1 heading rule); otherwise it lives in prose.

## Stage 3 — Gates (fixed order)

- **Gate 0 — structure & format check**: mechanical, no fact-sheet needed.
  - Every visual idea in the brief has a matching `[VISUAL: ...]` placeholder
    in the draft — none dropped, none added beyond the brief's list, and no
    more than 3 total.
  - Every `[CTA: ...]` placeholder's `/products/<handle>` or
    `/collections/<handle>` resolves to a real, currently-carried Shopify
    catalog entry; drop any that doesn't (never invent one to fill the slot).
  - Every FAQ question is H3 under the "FAQ" H2 — none promoted or demoted.
  - `meta.md` title ≤70 characters and includes the primary keyword;
    description ≤155 characters, includes the primary keyword, and ends with
    a CTA per the style guide; slug is lowercase/hyphenated with no query
    parameters or stop-word bloat.
  - Fix violations directly (they're mechanical) rather than sending the
    whole draft back to the writer; only loop back if a fix requires new
    brief input (e.g. no real product exists for a required CTA).
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

`draft.md` (with inline links, standard closing disclaimer, References),
`meta.md`, `inbound-links.md`, `gate-log.md`.
Ready for Gate 4 human medical + compliance review
(`config/compliance-checklist.md`). Never call it publishable before that.
