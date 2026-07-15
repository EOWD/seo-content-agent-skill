# SEO Content Engine

Claude-orchestrated draft pipeline for the infant nutrition content program.

**Architecture doc (living document):** source of truth is `docs/architecture.html`.
Published (readable) version: https://claude.ai/code/artifact/c251137b-d797-4c6a-baaa-13e9788230e7

To update the doc: edit `docs/architecture.html` (ask Claude — "update the
architecture doc: <what changed>"), then republish it to the same URL with the
Artifact tool, passing that URL as `url`. Republishing to that exact link only
works from the account that owns the artifact; from another account, publish a
new artifact and update this README with the new link.

**Scope (current):** produces review-ready drafts. No auto-publishing — the SEO
manager reviews, routes to the medical reviewer, and publishes manually.

## How to operate (SEO manager)

Open this folder in the Claude Code desktop app and use:

The commands follow the same steps an SEO writer would take for a cluster —
plan, research, approve, write, preview, review — with an approval checkpoint
before writing ever starts:

| Step | Command | What it does |
|---|---|---|
| 0 (once) | `/ingest-archive` | Load all old posts into the Pinecone KB + derive tone of voice from the full corpus |
| 1 | `/plan-cluster <topic>` | Keyword research → pillar + spoke plan → cannibalization check → **your approval** → briefs |
| 2 | `/research-cluster <slug>` | Linked research draft per article (claims + sources, news scan, fresh angles) → **your approval** → frozen fact sheets |
| 3 | `/run-cluster <slug>` | Parallel drafts from approved research → gates → link map → previews → review pack |
| 4 | `/preview-article <slug>` | Blog-styled HTML preview + Google SERP snippet preview (stable link, re-renders after edits) |
| 5 | `/write-article <slug>` | Single article, plus the prompt-edit loop ("shorten the intro") — edits are claim-diff protected |
| anytime | `/suggest-updates` | News + guideline sweep vs the archive → evidence-linked update and topic suggestions |
| meta | `/update-skill <name> <change>` | Improve any pipeline command — edits it AND ships a git branch + PR in one command |

Output lands in `clusters/<slug>/` — each article as `draft.md` with its
`fact-sheet.md`, `meta.md`, and a cluster-wide `link-map.md` and `review-pack.md`.

## Folder layout

```
config/            style guide, trusted-source allowlist, compliance checklist
clusters/<slug>/   briefs, drafts, fact sheets, review pack (one folder per cluster)
.claude/skills/    the pipeline commands
```

## Prerequisites

- `PINECONE_API_KEY` exported in the shell (knowledge base)
- Ahrefs MCP connected (keyword data; `/plan-cluster` degrades to manual keywords without it)
- Old blog archive ingested via `/ingest-archive`
