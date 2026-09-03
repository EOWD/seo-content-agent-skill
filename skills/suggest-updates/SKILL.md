---
name: suggest-updates
description: Scan recent news and guideline changes against the existing article archive and suggest which published posts need updates and which new topics are worth covering — every suggestion backed by a linked source. Use when the user says "suggest updates", "what should we update", "any news on our topics", or "/suggest-updates".
---

# Suggest updates and new topics — backed by linked evidence

No article is written or changed by this skill; it produces a prioritized,
evidence-linked suggestion list for the SEO manager.

## Steps

1. **Map the territory.** Pull the topic list from the `blog-archive` KB
   (main topics + publish dates of the newest post per topic).
2. **News sweep.** For each major topic area, search the last 6–12 months for:
   new or changed official guidance (WHO/AAP/NIH/CDC/NHS/EFSA/FDA), recalls
   or safety notices, major new studies, and shifts in expert consensus.
   Every finding: date + working source link. Discard anything that can't be
   linked to an allowlisted or clearly authoritative source.
3. **Match findings to the archive** (KB search): which published posts state
   something the news item changes, extends, or contradicts.
4. **Produce the suggestion list**, two sections, priority-ordered:
   - **Update existing posts** — post URL, what changed, the linked evidence,
     and the concrete edit suggestion (one line).
   - **New content opportunities** — topic, why now (linked evidence), and
     which cluster it would belong to.
5. Save to `suggestions/<year>-<month>.md` and present the top items to the
   user. Accepted update items become update-briefs; accepted new topics feed
   the next `/plan-cluster`.

## Rules

- A suggestion without a link is not a suggestion — drop it.
- Never edit published content or the KB from this skill.
- Note per item whether it's a compliance-relevant change (regulatory /
  safety) — those are flagged URGENT ahead of SEO-motivated updates.
