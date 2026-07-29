# Proven workflows + write etiquette

Recipes that work in real sessions, plus the etiquette that keeps writes
safe. Each maps to the tools named in SKILL.md's operating loop.

## The weekly audit

"Check all my feeds' health, group any errors by root cause, and tell me
the single highest-impact fix."
→ `spf_list_feeds` → `spf_feed_health` per feed → `spf_feed_findings`.
Findings arrive grouped by cause with affected counts — lead with the
config-level cause that collapses the most errors (one domain/mapping fix
often clears hundreds of per-row errors).

## Disapproval triage

"Google disapproved products. Separate config-level causes from per-product
ones, give me admin links for the manual fixes."
→ `spf_feed_findings` (severity: error) → for each category decide:
pattern (→ rule or mapping fix) vs per-product (→ overrides, or the
included Shopify admin links for source-data fixes like missing images).

## Title sweep (preview-first in action)

"Improve my titles following Google best practices."
→ `spf_feed_products` to sample current titles → draft a rule →
`spf_preview_rule` → SHOW THE USER THE DIFF → only then `spf_apply_rule`
with the returned token → `spf_run_feed` → verify a row with
`spf_debug_row`. Applying without a preview is structurally impossible —
the apply call requires the preview token for the exact rule text.

## One-product fix

"Fix this one product's brand/title/description in the feed (don't touch
Shopify)."
→ typed field: `spf_set_overrides` (undo: `spf_restore_overrides`).
→ arbitrary source cell: `spf_set_cells` with `preview: true` first; null
restores. Both report which output columns the edit feeds
(`propagates_to`).

## Campaign segmentation via custom labels

"Label products by price band / margin tier / season for bidding."
→ bulk pattern: rules writing `[G]custom_label_N`.
→ specific products: `spf_set_overrides` with `custom_labels`.
→ margin tiers specifically: use `spf_set_margin_tiers` (next recipe) — it
manages the tier rules as a set for you.
Labels never affect approval — safe to iterate.

## Margin tiers from broad guidance (no per-SKU cost needed)

"Skincare runs ~high margin, clearance is thin — bid by margin tier."
→ `spf_set_margin_tiers` by **category**: turn the merchant's broad
guidance into tiers keyed on a source field (product_type / vendor / tags)
written to a `custom_label` slot — e.g. `high_margin` when `product_type in
(Serums, Moisturizers)`. **No per-item cost data required** — this is the
common case, since most shops don't track cost. If the shop DOES track cost,
use **band** tiers over computed margin % instead; the response's
`cost_coverage` tells you how usable bands are (with_cost / total).
→ `spf_run_feed` (regenerate) → confirm a product with `spf_debug_row`,
which names the assigned margin tier (margin → label → slot). Idempotent:
re-running replaces the tiers; `clear: true` removes them. Margin-tier rules
are managed here, not in the generic rule tools.

## New channel launch

"What does channel X need that Google doesn't?"
→ `spf_channel_status` for what's connected → compare requirements
(see gmc-field-requirements.md and channel-quirks.md) → gap-check with
`spf_feed_products` searches for missing fields before connecting.

## Pre-holiday readiness

"Audit images, availability, prices, shipping across every channel;
prioritized fix list."
→ health + findings per feed → `spf_feed_products` spot-checks →
one prioritized list: config fixes first, then rules, then per-product.

## Write etiquette (always)

1. **Preview before apply** — rules require it (token); cells offer it
   (`preview: true`); use it whenever the user hasn't seen the change.
2. **Named changes only** — vague asks ("clean up my feed") get
   investigation and a proposal, not writes.
3. **Small batches** — overrides/restores take ≤25 products per call;
   chunk larger sets and say you're chunking.
4. **Close the loop** — after writes: `spf_run_feed` (regenerate) → poll
   status → verify with `spf_feed_health` / `spf_debug_row`. Report
   evidence, not intentions.
5. **Everything is auditable** — writes land in the shop's audit trail;
   tell the user what you changed and how to undo it (restore / null /
   deactivate).
6. **One sync at a time** — respect in-flight refusals; poll
   `spf_channel_status` instead of retrying.
