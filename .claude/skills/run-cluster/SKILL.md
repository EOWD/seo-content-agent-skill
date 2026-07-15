---
name: run-cluster
description: Produce all drafts for a cluster whose research is approved — parallel writers, gates, internal-link map, previews, and review pack. Use when the user says "run the cluster", "write the cluster", or "/run-cluster <slug>".
---

# Run a full cluster to review-ready drafts

Input: a cluster slug where `/plan-cluster` (plan approved) and
`/research-cluster` (research approved, `fact-sheet.md` frozen per article)
have both completed. Output: every article drafted, gated, previewed —
no publishing.

## Steps

1. **Verify readiness.** Every article folder needs an approved
   `fact-sheet.md`. Articles without one are listed and skipped — offer to
   run `/research-cluster` for them; never draft from unapproved research.
2. **Confirm scope.** Show the user which articles will run (default: all
   ready ones) and fan out.
3. **Draft in parallel.** Run the `write-article` flow per article
   (Stages 1–3: clean draft → weave citations → gates). Pillar first if link
   sequencing matters, otherwise all parallel.
4. **Collect.** Per-article status: gates passed / flagged. A flagged article
   never blocks the others.
5. **Internal-link map** (cluster sync point — only when all drafts exist).
   Build `link-map.md`: spoke ↔ pillar links, links into existing posts
   (targets via the `blog-archive` KB), and suggested anchor edits to OLD
   posts that should link to the new articles (applied manually at publish).
   Update every `draft.md` to match the map, then re-run its Gate 3 claim diff.
6. **Previews.** Render each article via the preview-article skill and build
   `preview-index.html` at the cluster root linking all previews + the link
   map. Give the user the index link.
7. **Review pack.** Write `review-pack.md`: article table (title, slug, word
   count, gate status, preview link), instructions for the medical reviewer
   (review `draft.md` against `fact-sheet.md` and
   `config/compliance-checklist.md`; verdict APPROVE / REVISE / HOLD per
   article), and the reminder that nothing publishes without Gate 4.

## After the run

Prompt edits happen per article via `/write-article`'s editing loop
(claim-diff protected), with `/preview-article` re-rendering to the same URL.
