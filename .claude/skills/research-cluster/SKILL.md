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
2. **Evidence research** — search the web and the paper literature for
   current guidance. Every claim recorded MUST carry a working link to an
   allowlisted source (`config/trusted-sources.md`), plus publication date
   and supporting quote. The web may be read freely; only allowlisted
   sources may be cited.

   **Paper search — run BOTH sources and merge** (load credentials first
   with `set -a; source .env; set +a` — see `config/kb.md`):

   a. **Elicit** (semantic — finds conceptually relevant work):
   ```
   curl -s -X POST https://elicit.com/api/v2/search/papers \
     -H "Authorization: Bearer $ELICIT_API_KEY" \
     -H "Content-Type: application/json" \
     -d '{"query": "<natural language question>",
          "searchMode": "semantic",
          "maxResults": 20,
          "filters": {"minYear": 2019,
                      "typeTags": ["RCT", "Meta-Analysis", "Systematic Review"]}}'
   ```
   b. **PubMed E-utilities** (keyword — guarantees exact clinical terms and
   the newest indexed papers):
   ```
   https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=<terms>&sort=relevance&retmax=20&retmode=json
   then efetch/esummary the returned PMIDs for titles, dates, journals
   ```
   Merge the two result sets, dedupe by DOI/PMID, and prefer: meta-analyses /
   systematic reviews / RCTs, then recency, then `citedByCount`.
   - Cite the PAPER (DOI link or pubmed.ncbi.nlm.nih.gov/<pmid>), never
     elicit.com or the search engine used.
   - Elicit rate limit: 100 requests/min — batch queries per article.
   - Gotchas (learned 2026-07-16): Elicit's WAF 403-blocks the default
     Python-urllib user agent — send a curl-like `User-Agent` header and
     space calls ~3s apart. PubMed E-utilities 429s on rapid calls — sleep
     ≥1s between esearch and esummary.
   - If `ELICIT_API_KEY` is missing, proceed with PubMed alone and note it
     in the research draft header.
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

## Approval checkpoint (mode-aware — see config/pipeline-settings.md)

6. **Present the research for approval — as an artifact.** Publish
   `clusters/<slug>/research-review.html` (load the artifact-design skill
   first): per article, the full claims table with every source link
   clickable, the news scan with dates, the suggested angles with their
   supporting evidence, and any conflicts between sources highlighted. This
   is built for an expert reviewer — make links spot-checkable and quotes
   verbatim. Give the manager the link; they approve, edit, or reject per
   article IN CHAT. In **auto** mode (default), publish the page as an
   audit trail, freeze the fact sheets, and proceed straight to drafting —
   the manager can interrupt anytime. In **checkpoint** mode, wait for their
   reply before freezing. Republish the page marked with the outcome.
7. When approved (or immediately, in auto mode), freeze the claims table of each `research-draft.md` into
   `fact-sheet.md` in the same folder — this is the writing contract that
   Gates 1 and 3 enforce. Unapproved articles do not proceed to drafting.

## Output

`clusters/<slug>/<article-slug>/research-draft.md` + `fact-sheet.md` for every
approved article; a note in `cluster-plan.md` recording approval date and any
articles held back.
