---
name: ingest-archive
description: One-time setup — load the full old-blog archive into the Pinecone knowledge base and derive the style guide from it. Use when the user says "ingest the archive", "load the old blogs", or "set up the knowledge base".
---

# Ingest the blog archive into the knowledge base

## Preconditions (check first, stop with clear instructions if missing)

0. `config/` exists in the working project. If not (fresh install of the
   plugin), copy the templates — `trusted-sources.md`,
   `compliance-checklist.md`, `style-guide.md` — from the plugin's own
   `config/` directory (`${CLAUDE_PLUGIN_ROOT}/config/`) into the project.
   All pipeline skills read `config/` from the working project.
1. `PINECONE_API_KEY` is set in the environment — a **write-capable** key
   (maintainer's). Also: the project `.claude/settings.json` denies Pinecone
   MCP write tools by default (KB protection — see `config/kb.md`). The
   maintainer must temporarily remove that deny block before ingesting and
   restore it afterwards. If the deny rules are active, stop and tell the
   user this — do not work around them another way.
2. A source for the archive, one of:
   - an export file the user provides (WordPress XML, CSV, or a folder of files), or
   - the site URL — discover posts via `/sitemap.xml` and fetch each one
     (WebFetch; use Firecrawl if pages need JS rendering).

## Steps

1. **Collect** every published post: URL, title, publish date, updated date,
   body text (strip nav/boilerplate), category/tags if present.
2. **Create the index** (once): use the Pinecone MCP `create-index-for-model`
   with an integrated embedding model; name: `blog-archive`; field map text
   field: `content`. If the index exists, skip.
3. **Chunk** each post: ~300–500 token chunks on paragraph boundaries, keeping
   heading context. Record for every chunk: `content`, `url`, `title`,
   `publish_date`, `section_heading`, `post_id`, `chunk_index`. Use this exact
   schema for every record — never vary field names.
4. **Upsert** all chunks via the Pinecone MCP in batches. Log counts:
   posts found / chunks written / failures. Never report success with silent
   gaps — list any post that failed.
5. **Verify**: run 3 sample `search-records` queries (a topic query, a title
   query, a tone query) and show the user the hits.
6. **Derive the tone of voice from the FULL corpus** (the initial
   tone-of-voice run — happens once, right after the KB is connected):
   - Quantitative pass over ALL posts: typical length, heading style,
     paragraph rhythm, reading level, how caregivers are addressed, hedging
     phrases, words the site always/never uses.
   - Deep-read 10–15 representative posts (spread across topics and years)
     for voice and structure patterns.
   - Fill in every TEMPLATE section of `config/style-guide.md`, including 3–5
     tone exemplar paragraphs with source URLs.
   Present the result to the user for correction — do not treat it as final
   until they confirm. Writer agents treat this file as authoritative from
   then on (plus retrieval-time exemplars from the KB per topic).

## Output

- Pinecone index `blog-archive` populated.
- `config/style-guide.md` filled in (pending user confirmation).
- A short ingest report: post count, chunk count, date range of corpus, failures.
