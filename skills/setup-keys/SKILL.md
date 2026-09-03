---
name: setup-keys
description: One-time setup of the API keys the pipeline needs (Pinecone, Elicit) — no terminal required. Use when the user says "set up my keys", "/setup-keys", "configure the API keys", or when a pipeline skill fails because a key is missing.
---

# Set up API keys (no terminal needed)

Gets a user from fresh install to working pipeline. Keys are stored in the
user's own Claude settings — NEVER in this repo, never committed, never in
plugin files.

## Steps

1. **Check what's already set**: for each of `PINECONE_API_KEY`,
   `ELICIT_API_KEY`, report set / not set (never print values).
2. **For each missing key, tell the user where to get it**, then ask them to
   paste it:
   - Pinecone (required): https://app.pinecone.io → API Keys → Create.
     **Operators (SEO manager) get their own key, never a shared one** — a
     per-person key can be revoked without breaking anyone else. If the plan
     tier offers custom key roles, create it READ-ONLY (Control plane:
     Read-only, Data plane: Read-only); on tiers without roles, the
     repo-committed `.claude/settings.json` deny rules block all Pinecone
     MCP write tools instead (do not remove them on operator machines).
     The pipeline only searches the KB, so everything works except
     `/ingest-archive` (maintainer-only). Never put the main project key on
     an operator's machine — the same Pinecone project holds the live
     shopping agent's indexes (`blog`, `blog-openai`, `products*`).
   - Elicit (optional — paper research; PubMed fallback works without it):
     https://elicit.com/settings → API. Requires a Pro plan or above.
     Key format starts with `elk_live_`.
3. **Store each provided key** in the user's `~/.claude/settings.json` under
   the `"env"` object (merge with existing content — read the file first,
   never clobber other settings):
   ```json
   { "env": { "PINECONE_API_KEY": "...", "ELICIT_API_KEY": "..." } }
   ```
   If the user prefers shell-level config, append `export KEY="..."` lines to
   their shell profile (`~/.zshrc` on macOS) instead — ask which they want
   only if they express a preference; default to settings.json.
4. **Ahrefs is not a key** — it's an OAuth login. Tell the user to type
   `/mcp`, select "claude.ai Ahrefs" (or the Ahrefs server if installed
   locally), and complete the browser login.
5. **Verify**:
   - Elicit: `curl` a 1-result paper search with the Bearer token; report
     ok / auth error (never echo the key).
   - Pinecone: the MCP server reads the env at session start — tell the user
     to restart Claude Code, then confirm with a `list-indexes` call in the
     new session.
6. Remind the user: keys are personal (or shared via the team password
   manager) — never paste them into repo files, commits, or shared docs.

## Security rules (absolute)

- Never write a key into any file inside the repo/plugin.
- Never print a key back in chat, logs, or previews.
- If a key is accidentally pasted into a file that git tracks, remove it and
  tell the user to rotate the key.
