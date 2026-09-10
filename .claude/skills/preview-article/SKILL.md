---
name: preview-article
description: Render a draft article as a blog-styled HTML preview (artifact) so the writer or SEO manager can see it as readers would — plus a Google SERP snippet preview of its meta tags. Use when the user says "preview the article", "/preview-article <article-slug>", "show me how it looks", or after prompt edits.
---

# Preview a draft as it would appear on the blog

Input: an article folder containing `draft.md` and `meta.md`.

## Steps

1. **Load the artifact-design skill first** (required before publishing any
   artifact), then build a single self-contained HTML page containing:
   - **SERP snippet preview** at the top: the meta title, URL slug, and meta
     description rendered the way Google shows them, with character-count
     warnings if title >70 chars (~600px) or description >155 chars (~960px),
     and a flag if the URL slug looks long/unclean (query parameters, stop-word
     bloat, or approaching 200+ characters).
   - **The article** rendered as a blog post: title, byline placeholder
     ("Medically reviewed by —, pending"), reading time, headings (FAQ
     questions rendered as H3), inline links (external links marked
     `target="_blank" rel="noopener"` with a small new-tab indicator, so the
     new-tab rule and first-link-internal rule are visually spot-checkable),
     `[VISUAL: ...]` placeholders rendered as a labeled image-placeholder box
     with its alt text visible, `[CTA: ...]` placeholders rendered as an
     actual styled button linking to the product/collection, and the
     References section.
   - Style it to approximate the live site's typography if known (check
     `config/style-guide.md` for site look notes); otherwise a clean,
     readable blog layout. Both light and dark theme.
   - A footer strip showing pipeline status: gates passed, word count,
     Gate 4 review still pending.
2. **Publish as a private artifact** and give the user the link. Use the file
   path `clusters/<slug>/<article-slug>/preview.html` so re-previewing the
   same article redeploys the same URL.
3. After each `/write-article` prompt edit, re-render to the same path so the
   preview link stays stable while the draft evolves.

## Notes

- This is a preview, not the publish step — no CMS, no live site. The
  artifact is private unless the user shares it.
- For a whole cluster, offer a `preview-index.html` at the cluster root
  linking all article previews plus the link map.
