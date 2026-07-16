---
name: plan-cluster
description: Plan a content cluster — keyword research, pillar + spoke structure, cannibalization check, and briefs. Use when the user says "plan a cluster", "plan content about X", or "/plan-cluster <topic>".
---

# Plan a content cluster

Input: a topic area (e.g., "weaning schedules"). Output: an approved cluster
plan with one brief per article in `clusters/<slug>/briefs/`.

## Steps

1. **Keyword research.** If the Ahrefs MCP is connected, pull keyword ideas,
   volumes, difficulty, and SERP features for the topic. If not connected,
   say so and ask the user to paste keywords (do not silently invent volumes).
2. **SERP scan.** For the top candidate keywords, WebSearch the live SERP:
   who ranks, what angle, what format (listicle/guide/FAQ), and visible gaps.
3. **Cluster design — size is the manager's call.**
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
4. **Cannibalization check.** For every proposed article, `search-records` the
   `blog-archive` Pinecone index for existing coverage. If an old post already
   targets that intent: mark the proposal as UPDATE (refresh the old post)
   instead of NEW, or drop it. Show what was found.
5. **Present the plan for approval — as an artifact.** Publish
   `clusters/<slug>/plan-review.html` (load the artifact-design skill first)
   containing, in this order:
   - **Keyword research table** — EVERY candidate keyword evaluated (not
     just the chosen ones): volume, difficulty, CPC, traffic potential,
     parent topic, intent, and SERP features (flag AI Overview presence).
     Mark each row chosen-as-primary / folded-into-<article> / discarded,
     with a one-word reason. Sortable columns if simple to do; at minimum
     sorted by volume. This is the manager's raw material — never show
     conclusions without the data behind them.
   - The plan table (article, primary keyword, intent, volume/difficulty,
     NEW vs UPDATE, overlapping old post).
   - SERP findings per keyword, and the cannibalization evidence —
     competitor titles and existing-post links clickable so the manager can
     spot-check everything from the page. Give
   them the link, then wait for their approval in chat (artifacts can't send
   clicks back — the page is the review surface, the reply is the approval).
   Apply their edits and republish the page so it records the approved state.
6. **Write briefs** (only after approval), one file per article in
   `clusters/<slug>/briefs/<article-slug>.md`:
   - primary keyword + secondary keywords
   - search intent and the reader's situation
   - working title + angle (why this beats what currently ranks)
   - outline (H2s), target word range
   - internal-link targets: 2–4 existing posts found via the KB, plus planned
     sibling articles in this cluster
   - notes from the SERP scan (format expectations, questions to answer)

## Output

`clusters/<slug>/briefs/` populated; a `cluster-plan.md` at the cluster root
recording the approved table and the date.
