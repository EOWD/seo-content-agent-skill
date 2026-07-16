# Handover — SEO Content Engine

**Date:** 16 July 2026 (supersedes the 15 July handover)
**Repo:** `/Users/eiad/Desktop/SEO-CO` — plugin `seo-content-engine` **v0.8.6**, installed user-wide
**Status:** OPERATIONAL. Pilot article produced end-to-end and validated. Second cluster mid-run. Not yet pushed to org git.

---

## 1. What this is

A Claude-orchestrated pipeline for **Organic's Best (organicsbestshop.com, Shopify)** that turns a topic into review-ready draft articles: keyword-driven cluster plan → linked research (every claim cited) → parallel writers → automated quality gates → HTML previews → review pack for human medical/compliance sign-off. **Drafts only — publishing is manual.**

- Living architecture doc: `docs/architecture.html` · published: https://claude.ai/code/artifact/c251137b-d797-4c6a-baaa-13e9788230e7
- The engine runs in the **Claude Code desktop app** (GUI, no terminal) or CLI. Everyone else (medical reviewer, stakeholders) only needs the artifact links in a browser.

## 2. Proof it works — the pilot (16 Jul)

"Parental Burnout: Symptoms and Expert-Backed Ways to Cope" went from topic → gated, cited, previewed 2,100-word article in one session:
- 69 papers found (Elicit + PubMed), 34-claim fact sheet, every claim with link + verbatim quote
- Gates caught real issues: 1 unsourced claim removed, 6 style-guide violations fixed, zero claim drift
- Article preview: https://claude.ai/code/artifact/dc75b8c1-d765-48e5-b421-ff10536ae654
- Run dashboard: https://claude.ai/code/artifact/e626ffca-c572-4188-87a4-152fc733a834
- **Still pending: Gate 4 human medical/compliance review before it can publish.**

## 3. In flight right now

Cluster `breastfeeding-postnatal-vitamins` (run from the desktop app):
- Plan approved: pillar "Postnatal Vitamins While Breastfeeding" (**7,700/mo, KD 7** — best keyword found to date) + Femibion 3 spoke (KD 0). Deferred articles A2–A8 recorded in `clusters/breastfeeding-postnatal-vitamins/cluster-plan.md`.
- Plan artifact: https://claude.ai/code/artifact/b395e203-3558-4d6c-90ed-19dc19e4e021
- **Next step: `/research-cluster breastfeeding-postnatal-vitamins`**
- ⚠️ Femibion 3 label facts: femibion.de is geo/bot-blocked (see memory note `femibion-label-verification`). **Since v0.8.6 the primary source for carried products is our own Shopify metafields** (`tabs.ingredients` etc.) — check the store's Femibion 3 product first; a human reading the physical pack is the fallback. Never publish the unverified cross-retailer values.

## 4. How the SEO manager operates it

Fully automated by default (`config/pipeline-settings.md`, `approval_mode: auto`) — artifacts are a live audit trail, not blockers. The manager can interrupt any run in plain language.

| Step | Command | Output |
|---|---|---|
| 0 (once) | `/ingest-archive` | ✅ DONE — 215 articles / 1,133 chunks in Pinecone `blog-archive`, tone-of-voice style guide derived |
| 1 | `/plan-cluster <topic> <n>` | Keyword table + plan artifact; asks size once if `<n>` omitted (the ONLY question in auto mode) |
| 2 | `/research-cluster <slug>` | Research-review artifact (claims + links + news scan), fact sheets freeze |
| 3 | `/run-cluster <slug>` | Parallel drafts → gates → link map → previews → dashboard → review pack |
| edits | plain language | "shorten the intro" — claim-diff protected; systematic gaps also fix the skill permanently |
| anytime | `/suggest-updates` | Evidence-linked update/topic suggestions from news vs the archive |
| meta | `/update-skill <name> <change>` | Edit + version bump + branch + PR in one command (needs org remote) |

## 5. Non-negotiable guardrails (encoded in skills — do not remove)

1. Writers have **no web access**; drafts may only contain claims from the frozen, human-reviewable fact sheet. Gate 1 machine-checks this; Gate 3 re-checks after every edit.
2. Medical citations only from `config/trusted-sources.md` (WHO, AAP, NIH/PubMed, CDC, NHS, EFSA, FDA, + apa.org for psychology). Papers cited by DOI/PMID.
3. **Product facts** (ingredients/dosage): our Shopify metafields first (carried products), manufacturer page fallback; metafield↔label mismatches are flagged as store-data bugs. Product sources never support health claims.
4. Old blog posts are tone/coverage/link material — never medical sources.
5. Gate order fixed: fact-check → humanizer (style-only) → claim diff → **human medical + compliance review (blocking, never automated)**. Compliance: WHO Code for formula content, supplement claim rules for vitamins (`config/compliance-checklist.md`).
6. Prevalence-style stats stay ranges; no effect-size overclaims; sensitive findings route to help, never shame.

## 6. Connections (all live)

| Component | Status |
|---|---|
| Pinecone KB (`blog-archive`, ns `articles`) | ✅ 1,133 chunks; details in `config/kb.md`. Keys now in `~/.claude/settings.json` env → MCP works in fresh sessions. NEVER write to `blog`, `blog-openai`, `products*` (live shopping agent's indexes) |
| Ahrefs | ✅ OAuth'd (Advanced plan, ~1M units/mo) |
| Elicit + PubMed | ✅ Dual paper search. Gotchas encoded in skill: curl-like User-Agent for Elicit, ≥1s spacing for PubMed |
| Shopify Storefront API | ✅ Articles + product metafields (`.env`) |
| Web search / fetch | ✅ Built-in |
| Surfer SEO | ⏸ Optional; internal SERP scorer active. If plan tier has API access, add `SURFER_API_KEY` to `.env` |

Credentials: `.env` in repo root (git-ignored) + user-level settings env. Never in repo files or chat.

## 7. Outstanding

1. **Push to org git** (e.g. `sghwtrade/seo-content-engine`, private) — activates the `/update-skill` PR flow and team installs (`/plugin marketplace add <org>/<repo>` + `/plugin install seo-content-engine@sghw-seo`; plugin commands run in the terminal surface, installs apply to the desktop app too).
2. **Gate 4 reviewer** — source a licensed medical reviewer; budget hours per cluster. Pilot article is waiting on this.
3. **Femibion label verification** (see §3) and finish the postnatal cluster: research → run.
4. **Templates for artifacts** (agreed direction, not yet built): `templates/` shells for dashboard/plan-review/research-review/preview + fill script — cuts the biggest per-run time cost.
5. **Onboarding guide** for the manager (planned, not yet written).
6. Style guide `config/style-guide.md` is corpus-derived and used by the pilot — still marked pending a final human read-through.
7. Deferred by design: publish automation, GSC feedback loop (architecture Phases 3–4).

## 8. Version trail (git log has full detail)

v0.1 plugin packaging → v0.2 Elicit → v0.3 setup-keys → v0.4 dual paper search → v0.5 SEO pass (Surfer/internal) → v0.6 run dashboards → v0.7 artifact review surfaces → v0.8.x auto mode, size question, keyword tables, ToC rule, self-improvement rule, supplement compliance, metafield product facts.

Rule of the build: **a correction given once is never needed twice** — fixes land in skills/config with a version bump, and (post-remote) ship to the team as PRs.
