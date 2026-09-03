# Knowledge base — connection details

All pipeline skills search this index. Credentials come from `.env` in the
project root — load with `set -a; source .env; set +a` before any API call.

| | |
|---|---|
| Pinecone index | `blog-archive` (integrated embeddings: llama-text-embed-v2, 1024d) |
| Namespace | `articles` |
| Text field | `content` (searches are plain text — Pinecone embeds internally) |
| Record schema | `_id` (`<handle>#<chunk_index>`), `content`, `title`, `url`, `publish_date`, `section_heading`, `post_id`, `chunk_index` |
| Source | Shopify Storefront API (`SHOPIFY_DOMAIN` + `SHOPIFY_STOREFRONT_TOKEN` in `.env`) |
| Last ingest | 2026-07-16 — 215 articles, 1,133 chunks (corpus 2020-07 → 2026-07-14) |

Re-ingest (after new posts are published): re-run `/ingest-archive` — it
re-fetches from Shopify and upserts by `_id`, so it's safe to repeat.

⚠️ Never write to the live agent's indexes in the same Pinecone project:
`blog`, `blog-openai`, `products`, `products-large` belong to the Organic's
Best shopping agent (server-v2). This pipeline reads/writes ONLY `blog-archive`.

Protection (all layers active since 2026-08-03):
1. **Deletion protection enabled on ALL five indexes** in the project —
   no index can be dropped until protection is explicitly disabled first.
2. **Agent-level write lock**: `.claude/settings.json` (committed) denies the
   Pinecone MCP write tools (`upsert-records`, `create-index*`,
   `delete-index`) for everyone working in this repo. Searches are
   unaffected. To re-run `/ingest-archive`, the maintainer temporarily
   removes the deny block, ingests, then restores it.
3. **Local snapshot**: `backups/blog-archive-<date>.jsonl` (git-ignored) —
   id + full metadata/content for every record. Plan-tier Pinecone backups
   are unavailable (requires Standard+); ultimate recovery is
   `/ingest-archive`, which fully rebuilds the KB from Shopify.

Key policy: per-person API keys, one per machine, so any key can be revoked
without touching the others (console → API Keys). If the plan tier allows
custom key roles, operator keys should be created **read-only** (Control
plane: Read-only, Data plane: Read-only — set at creation; permissions
cannot be edited later). On plan tiers without key roles, the agent-level
write lock above is the operator guardrail. The main/server key stays on
server-v2 and is never distributed.
