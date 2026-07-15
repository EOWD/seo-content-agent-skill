---
name: research-cluster
description: Produce a linked research draft for every article in an approved cluster plan — claims with source links, a news scan, and fresh-angle suggestions — then get the SEO manager's approval before any writing starts. Use when the user says "research the cluster", "/research-cluster <slug>", or after a cluster plan is approved.
---

# Research a cluster (approval gate before writing)

Input: `clusters/<slug>/briefs/` from an approved `/plan-cluster` run.
Output: one approved `research-draft.md` per article. **No article writing
happens in this skill** — research is reviewed and approved first.

Read `config/trusted-sources.md` before starting.

## Per article (parallel agents, one per brief)

1. **Coverage check** — query the `blog-archive` Pinecone index: what has the
   site already said on this topic? List overlapping claims and posts (these
   become internal-link targets and repetition-avoidance notes).
2. **Evidence research** — search the web and PubMed for current guidance.
   Every claim recorded MUST carry a working link to an allowlisted source
   (`config/trusted-sources.md`), plus publication date and supporting quote.
   The web may be read freely; only allowlisted sources may be cited.
3. **News scan** — search for developments from the last 12 months on this
   topic: new guidelines, regulatory changes, recalls, notable studies, shifts
   in official advice. Each item: date, source link, one-line relevance note.
4. **Fresh angles** — 2–3 suggested angles that would make this article more
   current or distinctive than what ranks today — each angle backed by a
   linked item from steps 2–3, never by speculation.
5. Write `clusters/<slug>/<article-slug>/research-draft.md`:

   ```
   ## Claims (the future fact sheet)
   | # | Claim | Source | Link | Date | Quote |
   ## What we've already published (KB)
   ## News scan (last 12 months)
   ## Suggested angles (each linked to evidence)
   ## Open questions / conflicts between sources
   ```

## Approval checkpoint (blocking)

6. Present the SEO manager a per-article summary: claim count, strongest news
   items, suggested angles, and any source conflicts. They approve, edit, or
   reject per article. Apply their edits.
7. On approval, freeze the claims table of each `research-draft.md` into
   `fact-sheet.md` in the same folder — this is the writing contract that
   Gates 1 and 3 enforce. Unapproved articles do not proceed to drafting.

## Output

`clusters/<slug>/<article-slug>/research-draft.md` + `fact-sheet.md` for every
approved article; a note in `cluster-plan.md` recording approval date and any
articles held back.
