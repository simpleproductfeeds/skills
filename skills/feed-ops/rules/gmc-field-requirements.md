# Google Merchant Center field rules the validator enforces

These are the constraints SPF's Google Shopping validator checks — the ones
that actually cause disapprovals and warnings. When `spf_feed_findings`
reports a category, this is the domain knowledge behind it.

## Identity

- `id` — required, unique per item, stable across runs (changing ids resets
  Google's item history; never remap `id` casually).
- `item_group_id` — groups variants of one product; required whenever
  variants share a product.
- **GTIN / MPN / brand**: provide `gtin` when the product has one (barcode).
  Without a GTIN, provide `brand` + `mpn`. When neither exists,
  `identifier_exists` must be `false` — SPF sets this automatically when
  GTIN is missing, which downgrades the issue to a warning, not an error.
  Fabricating GTINs causes disapprovals; prefer honest
  `identifier_exists=false`.

## URLs and images

- `link` and `image_link` must be absolute `http(s)://` URLs. The classic
  root cause for a whole-feed error wall is relative links (e.g.
  `?variant=123…`) from a store domain misconfiguration — one config fix,
  not per-product edits.
- Missing `image_link` is per-product (Shopify product has no image) — the
  findings include a Shopify admin deep link to fix at the source.

## Enumerated fields (exact values matter)

- `availability`: `in stock`, `out of stock`, `preorder`, `backorder`.
- `condition`: `new`, `refurbished`, `used`.
- `gender`: `male`, `female`, `unisex`; `age_group`: `newborn`, `infant`,
  `toddler`, `kids`, `adult`. SPF normalizes common variants at validation.

## Text

- `title` ≤ 150 characters (Google truncates ~70 in display — front-load
  what matters: brand, product type, key attributes).
- `description` ≤ 5000 characters; SPF truncates safely at transform.
- Avoid promotional text in titles ("free shipping", ALL CAPS) — policy
  disapprovals, not format errors.

## Categorization & pricing

- `google_product_category` — Google's taxonomy; SPF can auto-fill blanks
  via AI categorization when the feed has it enabled.
- `price` must match the landing page and include currency handling; sale
  pricing uses `sale_price` (+ optional effective date range).

## Custom labels

- `custom_label_0` … `custom_label_4` — five free-text slots for campaign
  segmentation (bidding tiers, margin bands, seasonality). They never
  affect approval; they exist for the merchant's Ads structure. Writable
  per-product via `spf_set_overrides` (`custom_labels`) or in bulk via
  rules.
