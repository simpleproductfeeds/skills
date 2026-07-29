# Pipeline precedence — how a Shopify row becomes a feed row

Every SPF feed row is produced by one canonical pipeline. Knowing the order
explains every "why does this value look like that" question — and
`spf_debug_row` shows this exact journey for any row.

## The order (what runs before what)

1. **Product selection** — rows outside the feed's selection are dropped.
2. **Cell edits** (the in-app Source Data spreadsheet and `spf_set_cells`)
   are overlaid onto the **source** row — before mapping, keyed by source
   column name.
3. **Column mapping** — each feed output column reads from its mapped source
   column (`brand ← vendor`, `gtin ← barcode`, `id ← sku`, … overridable via
   `spf_update_mappings`).
4. **Product overrides** (`spf_set_overrides`) — typed per-product values
   (title, description, brand, GTIN, MPN, condition, exclusion, custom
   labels) merge in.
5. **Translations** (language feeds) and **AI categorization** (fills a
   blank google_product_category when enabled).
6. **Merchant rules** — the feed's rule list (exclude / set / replace)
   applies, in order. **Margin-tier custom labels ride here**: they are
   ordinary set-rules over a computed margin field, but managed as a set by
   `spf_set_margin_tiers` (not the generic rule tools).
7. **Validation + channel rendering** — normalization (gender/category),
   then the channel renderer produces the final output format.

## What wins over what (the precedence traps)

- **Rules beat cell edits.** A rule that writes a column overrides a cell
  edit on the same column. If a user's cell edit "isn't sticking," check
  the rules first — `spf_debug_row` will show the rule responsible.
- **Cell edits are SOURCE overlays.** They change what mapping reads, so an
  edit to `vendor` flows to whatever output column reads vendor (usually
  `brand`). Writing a feed-output name as a cell does nothing — the tool
  rejects it with the correct source column named.
- **Rule fields have two prefixes**: `[D]column` reads the SOURCE row
  (before mapping); `[G]column` reads the mapped OUTPUT row. Example DSL:
  `Exclude when [D]title contains Sample` ·
  `Set [G]title to NEW | + [D]title`.
- **Some `[D]` fields are computed, not raw source columns.**
  `canonical_margin_pct` (margin % derived from cost + price) is usable in
  rule conditions and by `spf_set_margin_tiers`, even though it is not a
  spreadsheet/grid column. It reads nil when a product has no cost data.

## Known sharp edges

- Feed `id` and `mpn` commonly both read source `sku` — you cannot give
  them different values through cell edits; use a typed override (`mpn`) or
  a rule instead.
- A few output columns are computed by the channel renderer from normalized
  canonical data (e.g. `shipping_weight` from a canonical weight) — cell
  edits to those source columns may not change the rendered output. Trust
  `spf_debug_row`'s rendered view; if the preview shows no change, that
  column is renderer-owned.
- Exclusion reasons are explicit: a row excluded by selection, a rule, an
  override, or the shop's plan allowance says so in `spf_debug_row` /
  `spf_feed_products`. "Plan allowance (N) reached" means the catalog is
  larger than the subscription tier — an upgrade conversation, not a bug.
- After any write, validation counts recompute automatically (responses
  say `revalidation_triggered` / `revalidating`) — re-read health after a
  minute rather than assuming stale counts.
