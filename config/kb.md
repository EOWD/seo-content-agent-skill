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
