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
  When no GTIN is assigned, use the applicable `brand` + manufacturer-assigned
  `mpn`; a blank feed cell does not establish that no identifier exists.
  Set `identifier_exists=false` only when you have confirmed the product has
  no assigned unique product identifiers. Check supplier/manufacturer evidence
  and the rendered row before recommending a change. Do not invent GTINs or
  substitute an arbitrary store SKU for a manufacturer-assigned MPN.
  Any SPF default or local warning severity is not evidence that Google's
  requirements are met: if a rendered row has `identifier_exists=false` merely
  because GTIN is missing, diagnose and correct the actual identifier data.
  See Google's current [identifier_exists requirements](https://support.google.com/merchants/answer/6324478)
  and [incorrect product identifier guidance](https://support.google.com/merchants/answer/12158126).

### Review checks (synthetic, not live Merchant Center tests)

- Manufacturer assigned a GTIN, but Shopify barcode is blank: recover the
  real GTIN and fix its mapping; do not set `identifier_exists=false`.
- No GTIN assigned, but manufacturer assigned an MPN and brand: preserve
  those identifiers; missing GTIN alone does not justify a false value.
- Custom item confirmed to have no assigned identifiers: a false value is
  appropriate. An unknown supplier record remains a research gap, not proof
  that identifiers do not exist.

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
