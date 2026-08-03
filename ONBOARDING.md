# SEO Content Engine — Installation & Onboarding Guide

For the SEO manager of **Organic's Best** (organicsbestshop.com).
The engine turns a topic into review-ready draft articles: keyword-driven
cluster plan → cited research → parallel drafts → automated quality gates →
HTML previews → review pack. **Drafts only — publishing stays manual, and
every article needs human medical/compliance sign-off (Gate 4) before it can
go live.**

---

## Part 1 — Installation (one-time, ~20 minutes, done with the maintainer)

You need: the **Claude Code desktop app** installed and signed in with your
work account, plus the project folder handed over by the maintainer.

### 1. Get the project folder

The maintainer gives you a zip (e.g. `SEO-CO-install.zip`). Unzip it to:

```
~/Desktop/SEO-CO
```

Any location works — the rest of this guide assumes this one. The zip
contains **no credentials**; those come separately in steps 3–4.

### 2. Install the plugin

Open a terminal, run `claude` (any folder), then run these two commands
inside the session:

```
/plugin marketplace add ~/Desktop/SEO-CO
/plugin install seo-content-engine@sghw-seo
```

Quit and reopen Claude Code. The plugin is now available everywhere,
including the desktop app.

### 3. Create your `.env`

In the `SEO-CO` folder, duplicate `.env.example` and rename the copy `.env`.
Fill in the values from the team password manager:

- `SHOPIFY_STOREFRONT_TOKEN` — shared team value.
- `PINECONE_API_KEY` — **your personal key** (the maintainer creates one for
  you in the Pinecone console; it is yours alone and can be revoked without
  affecting anyone else).
- `ELICIT_API_KEY` — optional; paper research falls back to PubMed without it.

### 4. Store the keys for Claude

Open the `SEO-CO` folder in Claude Code (desktop app: Open Folder) and type:

```
/setup-keys
```

Paste the same Pinecone (and optional Elicit) key when asked. Restart the
session afterwards so the Pinecone connection picks up the key.

### 5. Connect Ahrefs

In a session, type `/mcp`, choose the Ahrefs server, and complete the
browser login with the team Ahrefs account.

### 6. Verify

In a fresh session in the `SEO-CO` folder, ask:

> search the blog archive for "goat milk formula"

You should get back chunks of old blog articles with titles and URLs. If you
do, the installation is complete.

---

## Part 2 — Daily operation

Everything runs in plain language or with these commands, in a session
opened in the `SEO-CO` folder. The pipeline is fully automated by default;
artifacts (links Claude gives you) are a live audit trail you can open in
the browser at any time. You can interrupt any run in plain language.

| Step | Command | What you get |
|---|---|---|
| 1 | `/plan-cluster <topic> <n>` | Keyword research table + cluster plan artifact |
| 2 | `/research-cluster <slug>` | Cited research for every article, for your approval |
| 3 | `/run-cluster <slug>` | Drafts → quality gates → previews → review pack |
| edits | plain language | "shorten the intro", "make the tone warmer" — claim-protected |
| anytime | `/suggest-updates` | Evidence-linked update ideas from news vs the archive |

A single article: `/write-article <article-slug>` · preview any draft:
`/preview-article <article-slug>`.

## Part 3 — Rules that are not yours to bend

These are enforced by the pipeline; if something seems blocked, this is
probably why. Do not work around them.

1. **Nothing publishes automatically.** Every article stops at Gate 4
   (human medical + compliance review).
2. **Writers can only use claims from the approved fact sheet** — if a claim
   is missing, it goes through research + your approval, not straight into
   the draft.
3. **Medical sources** come only from the trusted list (WHO, AAP, NIH, CDC,
   NHS, EFSA, FDA…). Old blog posts are tone material, never sources.
4. **Product facts** (ingredients/dosage) come from our own store data;
   mismatches get flagged, never guessed.
5. **The knowledge base is read-only from this machine.** The project
   settings deliberately block database writes (`.claude/settings.json` —
   do not edit or delete it). Re-ingesting the archive is a maintainer task.
6. Never touch the Pinecone indexes named `blog`, `blog-openai`,
   `products`, `products-large` — they belong to the live shopping agent.

## Troubleshooting

- **Pinecone errors / "no index found"** — restart the session; if it
  persists, re-run `/setup-keys`.
- **"Permission denied" on a database write** — working as intended (rule 5
  above). Tell the maintainer if you believe a re-ingest is needed.
- **Ahrefs not responding** — `/mcp` → re-login.
- **Anything else** — ask Claude to explain what failed; screenshots of the
  session to the maintainer.

Maintainer: Eiad Oraby (eiad.oraby@sghwtrade.com).
