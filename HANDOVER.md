# Handover — SEO Content Engine

**Date:** 15 July 2026
**Project folder:** `SEO-CO/`
**Status:** Architecture finalized and scaffolded. Not yet run — blocked on three inputs (see "Do these first").

---

## 1. What this is

A Claude-orchestrated pipeline that produces **review-ready draft articles** for a baby formula / infant-feeding website. This is YMYL medical content, so the design is built around verifiable sourcing and human medical review — not just writing speed.

**Current scope: drafts only.** The pipeline stops at a folder of finished drafts with citations, meta tags, and an internal-link plan. Publishing is manual. Auto-publishing and the performance feedback loop are designed (Phases 3–4 of the roadmap) but deliberately deferred.

**Full architecture (living document):**
- Source of truth: `docs/architecture.html` in this folder
- Published view: https://claude.ai/code/artifact/c251137b-d797-4c6a-baaa-13e9788230e7
- To change it: tell Claude "update the architecture doc: …" — the `/update-architecture` skill edits the file and republishes. Note: republishing to that exact URL only works from the account that owns the artifact; from another account the skill publishes a new link and updates this README/skill automatically.

## 2. Who does what

| Person | Tool | Role |
|---|---|---|
| SEO manager | **Claude Code desktop app** (GUI, no terminal) opened on this folder | Operates the pipeline via the skills below; approves cluster plans; routes drafts to review |
| Writers (ad-hoc work) | Claude Desktop Projects, connected to the same Pinecone KB | One-off articles, rewrites, brief refinement |
| Medical reviewer | Claude Desktop or plain documents | Gate 4: reviews `draft.md` against `fact-sheet.md` + `config/compliance-checklist.md`; verdict APPROVE / REVISE / HOLD |

## 3. What's been built

```
SEO-CO/
├── README.md                       operating guide (short version of this doc)
├── HANDOVER.md                     this file
├── docs/architecture.html          the living architecture document
├── config/
│   ├── trusted-sources.md          citation allowlist (WHO, AAP, NIH/PubMed, CDC, NHS, EFSA, FDA) + rules
│   ├── compliance-checklist.md     WHO Code checks the human reviewer runs per article
│   └── style-guide.md              TEMPLATE — auto-filled from the old blogs during ingest
├── clusters/                       output: one folder per content cluster (created on first run)
├── .claude-plugin/                 plugin + marketplace manifests — this repo installs org-wide
│                                   via /plugin marketplace add + /plugin install (see README)
├── .mcp.json                       Pinecone MCP config (travels with the plugin)
└── skills/
    ├── ingest-archive/             one-time: old blogs → Pinecone KB + tone of voice from full corpus
    ├── plan-cluster/               keywords → pillar+spoke plan → cannibalization check → briefs
    ├── research-cluster/           linked research drafts + news scan → approval → frozen fact sheets
    ├── write-article/              clean draft → weave citations → Gates 1–3 → prompt-edit loop
    ├── run-cluster/                all approved articles in parallel → link map → previews → review pack
    ├── preview-article/            blog-styled HTML preview + SERP snippet preview (stable link)
    ├── suggest-updates/            news vs archive → evidence-linked update/topic suggestions
    ├── update-skill/               edit a skill + branch + PR + version bump in one command
    └── update-architecture/        edit + republish the architecture doc
```

### The pipeline in one paragraph

`/plan-cluster <topic>` pulls keyword data, designs a pillar + up to 9 spokes, checks every proposed article against the existing site for cannibalization, and **stops for the SEO manager's approval** before writing briefs. `/run-cluster <slug>` then fans out one writer per brief in parallel. Each article goes: research agent builds a **fact sheet** (every claim cited to the allowlist) → writer drafts from brief + fact sheet + tone retrieved from the KB → **Gate 1** auto fact-check (every claim must map to the fact sheet) → **Gate 2** humanizer (style only, claims locked) → **Gate 3** claim diff (verifies the humanizer changed wording, not meaning). When all articles land, a cluster-wide internal-link map is built and a `review-pack.md` is generated for the medical reviewer (**Gate 4** — the only human gate, and the only blocking one).

### Non-negotiable guardrails (encoded in the skills — don't remove)

1. **Writers have no web access.** They can only use claims present in the fact sheet. This is what makes the fact-check gate enforceable.
2. **Medical citations only from the allowlist** in `config/trusted-sources.md`. The web may be *read* freely; it may not be *cited*.
3. **Old blogs are not a trusted source.** The Pinecone KB provides tone, coverage awareness, and link targets — but a claim found only in an old post still needs a fresh allowlisted citation.
4. **Gate order never changes**: fact-check → humanizer → claim diff → human review. The reviewer must see the final text.
5. **Nothing is "publishable" until Gate 4 approval.** Compliance ambiguity = HOLD, not "ship with notes." Infant formula marketing is regulated (WHO Code + national law); this is the one failure mode worse than losing rankings.

## 4. Do these first (in order)

1. **Pinecone API key** — create at https://app.pinecone.io, then in a terminal:
   `export PINECONE_API_KEY="your-key"` (add it to `~/.zshrc` to persist), restart the Claude Code session. *Blocks the knowledge base — hard prerequisite.*
2. **Authenticate Ahrefs** — in Claude Code, type `/mcp` and select "claude.ai Ahrefs", complete the browser login. *Without it, `/plan-cluster` asks for keywords manually instead of pulling volumes.*
3. **Provide the site domain** — Claude pulls `sitemap.xml` from it; that's the input to ingestion and the cannibalization check.
4. Run **`/ingest-archive`** — loads all old posts into Pinecone and drafts `config/style-guide.md` from the corpus. **Review and correct the generated style guide** — writers treat it as authoritative.
5. Pilot: **`/plan-cluster <topic>`** for one topic → approve the plan → **`/write-article`** on ONE brief first. Check the draft and fact sheet quality before trusting the fan-out.
6. First full run: **`/run-cluster <slug>`** → hand `review-pack.md` to the medical reviewer.

## 5. What's connected / what's not

| Component | Status |
|---|---|
| Web search + page fetch | ✅ Built into Claude Code — powers SERP scans and research |
| PubMed | ✅ Free E-utilities API via built-in fetch — no key needed |
| Elicit (paper research) | 🔑 Set `ELICIT_API_KEY` in the shell — research-cluster uses it automatically, falls back to PubMed without it |
| Pinecone (vector KB) | ❌ Needs `PINECONE_API_KEY` (step 1 above) |
| Ahrefs (keyword data) | ❌ Needs `/mcp` authentication (step 2 above) |
| Firecrawl (headless browser) | ⏸ Only if the site's pages don't render on plain fetch — test during first sitemap pull |
| Surfer SEO | ⏸ Optional scoring layer — decision deferred |
| CMS / publishing | 🚫 Out of scope by design (drafts only) |

## 6. Open decisions (recorded in the architecture doc)

- **CMS platform** (WordPress vs Shopify) — only matters when publishing automation (Phase 3) starts.
- **Medical reviewer sourcing** — staff vs contracted pediatric RD/RN. Gate 4 is the pipeline's rate limiter; budget reviewer hours **per cluster**, not per article.
- **Target markets** — determines which formula-marketing laws Gate 4 checks and whether hreflang is needed later.

## 7. Known sharp edges

- The skills are **v0** — expect to tune prompts after the first real cluster (especially fact-sheet granularity and the humanizer's restraint).
- Ten parallel writers consume significant usage; run the single-article pilot (step 5) before the first fan-out.
- The generated style guide is a *proposal* from the corpus — an hour of human correction there pays for itself across every future article.
- E-E-A-T site infrastructure (author bios, reviewer bylines on-page, editorial policy page) is assumed to exist site-side; the pipeline can't create it. Without it, YMYL content underperforms regardless of draft quality.
