---
name: plan-cluster
description: Plan a content cluster — keyword research, pillar + spoke structure, cannibalization check, and briefs. Use when the user says "plan a cluster", "plan content about X", or "/plan-cluster <topic>".
---

# Plan a content cluster

Input: a topic area (e.g., "weaning schedules") **or a product bundle
name/handle**. Output: an approved cluster plan with one brief per article in
`clusters/<slug>/briefs/`.

## Hard rules — keyword research (do not skip or soften these)

- **Exact volumes only.** Use the search volume Ahrefs returns for each
  keyword, verbatim. Never average, round, blend, or recalculate a volume
  across variants or endpoints.
- **No rank-checking at fetch time.** While pulling keyword data, do not
  evaluate whether the site already ranks for a keyword on another page
  (collection, product, or otherwise). That is not a reason to touch a
  keyword at this stage — ranking status is surfaced later, in review, never
  used to drop a keyword from the research list.
- **Off-catalog and irrelevant exclusion only.** The only keywords excluded
  during research are: (a) queries naming a brand/product the company does
  not currently carry (check the Shopify catalog if unsure — don't guess),
  and (b) queries with no relevant connection to the topic (e.g. "formula
  for dogs" on a baby-formula cluster). Nothing else gets dropped here — not
  duplicate phrasing, not a pack-size variant, not an exact branded match,
  not a keyword the site already ranks for elsewhere.
- **Higher-volume phrasing is flagged, not exclusive.** When the same search
  intent appears as two word-order variants (e.g. "HiPP formula stage 1" SV
  1.4K vs "HiPP stage 1 formula" SV 150), check for this on every keyword and
  note the pairing — but keep BOTH phrasings in the final candidate list. Flag
  the higher-volume one as the stronger pick for a primary keyword or heading,
  never drop the lower-volume variant from the list. The manager decides
  whether to use one, both, or neither.
- **Zero-SV keywords are limited, not banned.** Carry forward only a handful
  of zero-search-volume keywords — ones clearly relevant to the topic or
  well suited to an FAQ section. Do not include every zero-SV term.
- **100/mo minimum to anchor a topic.** A keyword under 100/mo search volume
  (this includes zero-SV) can never be a primary keyword or a standalone
  article topic — it doesn't carry enough demand to lead a page. It can
  still be used as a secondary keyword, surfaced in a subheading (H2/H3) or
  FAQ item of an article anchored by a keyword that does clear the bar. When
  designing the cluster in step 3, if a candidate topic's best representative
  keyword is under 100/mo, don't propose it as its own article — fold it in
  as a subheading of the most related topic instead.
- **The manager selects, not the research pass.** Keyword research produces
  the full candidate list; it does not narrow it. Narrowing happens once, in
  step 5, when the manager picks which secondary keywords each article
  actually uses.
- **Article assignment is the manager's call too.** When a keyword could
  plausibly serve more than one article (e.g. a brand-specific term that
  could anchor its own guide but also fits naturally in an overview
  article's prose), do not unilaterally decide it belongs to only one
  article to avoid keyword overlap between articles — that overlap
  judgment is the manager's, not the assistant's. Offer the keyword as a
  candidate under every article it could plausibly fit; flag the overlap
  if it's useful context, but never omit a keyword from another plausible
  article's candidate list on your own judgment. The manager decides final
  assignment, including using the same keyword across more than one
  article.

## Steps

0. **Bundle-sourced topics (only when the input is a bundle).** Pull the
   bundle's constituent products via the Shopify Storefront API and derive
   candidate topics from what problem each product solves (e.g. a "Reflux
   Relief Bundle" → topics on reflux symptoms, formula choice, feeding
   position). Treat each candidate the same as a topic area from here on.
1. **Keyword research.** If the Ahrefs MCP is connected, pull keyword ideas
   from all three angles — `keywords-explorer-matching-terms`,
   `keywords-explorer-related-terms`, and `keywords-explorer-search-suggestions`
   — with volume, difficulty, CPC, and SERP features for each. When the topic
   has more than one natural seed keyword (e.g. "stage 1 baby formula" and
   "stage 1 formula"), run the expansion separately for **each** seed as its
   own keyword group — do not limit the sweep to a single seed and treat the
   rest as its variants. Apply the hard rules above while building this list.
   For every article, settle on exactly one **primary keyword** (highest-volume
   term matching its intent, at least 100/mo — never zero-SV and never under
   the 100/mo anchor threshold) and a pool of candidate **secondary
   keywords** from the same endpoints that the article could cover,
   including sub-100/mo terms earmarked for subheadings rather than the
   topic itself — the manager narrows this pool to the actual secondary set
   in step 5. If Ahrefs isn't connected, say so and ask the user to paste
   keywords (do not silently invent volumes).
2. **SERP scan.** For the top candidate keywords, WebSearch the live SERP:
   who ranks, what angle, what format (listicle/guide/FAQ), and visible gaps.
3. **Cluster design — size is the manager's call.**
   - First, list **every distinct topic idea** the keyword research surfaced
     — not just the ones you're about to recommend. Each gets a short name,
     its main/representative keyword, and that keyword's search volume. This
     is the full menu the manager chooses from, not your shortlist — include
     ideas you're about to discard or merge, so the manager can overrule that
     call with the same data you had.
   - If the command included a count ("/plan-cluster weaning 6"), design to
     that size: 1 pillar + (n−1) spokes.
   - Otherwise, derive the natural size from the data — one article per
     distinct parent topic worth ranking for (don't pad; don't split one
     intent into two articles) — and ask the manager ONE question before
     fanning out: "The data supports N articles (pillar + spokes listed
     below) — how many do you want?" This question is asked even in auto
     mode, because it sets the cost of the run; it is the only question an
     auto run asks.
   Each article gets exactly one primary keyword and a distinct search
   intent — if two candidate articles answer the same intent, merge them.
   Per the 100/mo anchor threshold above, a topic idea whose best
   representative keyword is under 100/mo doesn't qualify as its own
   article — list it in the full menu as usual, but mark it folded into the
   most related topic (as a subheading candidate) rather than proposing it
   as a pillar or spoke.
4. **Cannibalization check.** For every proposed *article* (not individual
   keyword), `search-records` the `blog-archive` Pinecone index for existing
   coverage. If an old post already targets that intent: mark the proposal
   as UPDATE (refresh the old post) instead of NEW. This decides NEW vs
   UPDATE at the article level only — per the hard rules above, it is never
   a reason to remove a keyword from the candidate list; a keyword the site
   already ranks for elsewhere still belongs in the list for the manager to
   see and potentially select in step 5.
5. **Present the plan for approval, and let the manager select keywords —
   as an artifact.** Publish `clusters/<slug>/plan-review.html` (load the
   artifact-design skill first) containing, in this order:
   - **All topic ideas considered** — the full menu from step 3: topic name,
     main keyword, volume, and status (selected / merged into another topic /
     discarded), one-line reason for each. This sits above the narrowed plan
     table so the manager sees every option, not just the shortlist.
   - **Keyword research table** — EVERY candidate keyword from step 1, with
     its exact Ahrefs volume, difficulty, CPC, traffic potential, parent
     topic, intent, and SERP features (flag AI Overview presence). Mark each
     row chosen-as-primary / candidate-secondary-for-<article> / off-catalog
     (excluded) / irrelevant (excluded) — no other exclusion reason is valid
     per the hard rules. Sortable columns if simple to do; at minimum sorted
     by volume. This is the manager's raw material — never show conclusions
     without the data behind them.
   - The plan table (article, primary keyword, intent, volume/difficulty,
     NEW vs UPDATE, overlapping old post).
   - SERP findings per keyword, and the cannibalization evidence —
     competitor titles and existing-post links clickable so the manager can
     spot-check everything from the page.
   - A **secondary-keyword selection checklist per article** — every
     candidate secondary keyword for that article, so the manager can mark
     which ones to actually use. When a keyword plausibly fits more than one
     article, list it under every article it fits rather than picking just
     one — say plainly on the page that the grouping is a suggested
     starting point, not a restriction, and that the manager can reassign or
     duplicate any keyword across articles. Give them the link, then wait in
     chat for both approvals: the cluster plan itself, and which secondary
     keywords each article should use. Any candidate keyword the manager
     doesn't select is excluded from that article's brief — it stays visible
     in the record but isn't written in. Apply their edits/selections and
     republish the page so it records the approved state.
6. **Write briefs** (only after both approvals), one file per article in
   `clusters/<slug>/briefs/<article-slug>.md`:
   - primary keyword + the manager's selected secondary keywords only
   - search intent and the reader's situation
   - working title + angle (why this beats what currently ranks)
   - outline (H2s, with H3/H4 sub-points where the topic has natural
     sub-structure — e.g. comparison specs, step lists), target word range
   - visual ideas: any comparison-table or infographic concepts worth
     building (e.g. brand comparison table, dosage/age chart), each paired
     with an alt-text keyword to target
   - internal-link targets: 2–4 existing posts found via the KB, plus planned
     sibling articles in this cluster
   - notes from the SERP scan (format expectations, questions to answer)
   - keyword-usage note: selected secondary keywords — including ones the
     site already ranks for elsewhere — are used naturally in subheadings
     (H2/H3) and as anchor text linking to the relevant product/collection
     page. Don't force or overuse them; the primary keyword stays the
     article's main focus, secondaries appear only where they fit naturally.

## Output

`clusters/<slug>/briefs/` populated; a `cluster-plan.md` at the cluster root
recording the approved table, the manager's keyword selections, and the date.
