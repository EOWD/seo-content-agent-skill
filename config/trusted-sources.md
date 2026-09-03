# Trusted-source allowlist

Every medical or nutritional claim in a fact sheet MUST cite one of these
sources. General-web sources may inform background understanding but may
never be the citation for a claim.

## Allowlisted domains

| Source | Domain(s) |
|---|---|
| World Health Organization | who.int |
| American Academy of Pediatrics | aap.org, publications.aap.org, healthychildren.org |
| National Institutes of Health / PubMed | nih.gov, pubmed.ncbi.nlm.nih.gov, medlineplus.gov |
| Centers for Disease Control | cdc.gov |
| UK National Health Service | nhs.uk |
| European Food Safety Authority | efsa.europa.eu |
| US Food & Drug Administration | fda.gov |
| American Psychological Association (psychology/mental-health topics) | apa.org |
| Peer-reviewed papers via Elicit API or PubMed | (cite the paper's DOI or PubMed link — never elicit.com itself) |

Extend this table per target market (national health authorities) — additions
require SEO manager sign-off.

## Product-fact sources (separate class — added 2026-07-16)

For articles about specific products (e.g., Femibion, HiPP, Holle), product
facts — ingredients, dosages, usage instructions, age ranges — come from
structured product data, in this order:

1. **Our Shopify product metafields (primary, for products we sell).** The
   store maintains label data as metafields — the same data the live site
   and the AI shopping agent use. Pull via the Storefront API
   (credentials in `.env`), product by handle, metafields namespace `tabs`:
   `ingredients`, `nutritionfacts`, `nutritionfactsnote`,
   `preparation_steps`, `preparation_notes`, `feedingtable`; plus
   `info.stage` for age range. Record the retrieval date in the fact sheet.
2. **The official manufacturer product page** — for products we don't carry,
   or to cross-check when a metafield looks stale or is missing. Attribute
   as "according to the manufacturer".

Bounds (apply to both):
- Product sources support PRODUCT facts only — never health-benefit or
  medical claims. "Contains 200 mg DHA" → metafield/manufacturer is fine.
  "DHA supports infant brain development" → allowlist/peer-reviewed only.
- If metafield and manufacturer label disagree, flag it to the SEO manager —
  that's a store-data bug worth fixing, not a judgment call for the writer.

## Citation rules

1. One claim per fact-sheet line, with: source URL, publication date, and the
   supporting quote or data point.
2. Prefer sources published or reviewed within the last 5 years; flag anything
   older.
3. If two allowlisted sources disagree, record both — the article must reflect
   the uncertainty, not pick a side silently.
4. A claim that cannot be sourced from this list does not go in the fact sheet,
   and therefore cannot appear in a draft.
