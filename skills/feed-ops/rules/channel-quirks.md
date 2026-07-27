# Channel delivery quirks

"Sync" means something different per channel. `spf_channel_status` shows
connection state and recent sync logs; `spf_sync_channel` triggers delivery.

## Google Merchant Center (pull + fetch-now)

- Google fetches the feed URL on its own schedule (scheduled datafeed
  fetches configured in the app). `spf_sync_channel` for Google issues a
  **fetch-now** request — useful right after a fix so Google doesn't wait
  for the next scheduled fetch.
- Regenerating the feed (`spf_run_feed`) rewrites the hosted feed file;
  Google sees changes at its next fetch (or fetch-now).
- Live GMC-side processing status (what Google itself reports per datafeed)
  is on the REST API, not the MCP tools.

## Commission Junction (SFTP push)

- SPF uploads the file to CJ over SFTP. Files must land in CJ's
  `outgoing/` directory — files at the SFTP root are silently discarded by
  CJ within about a minute (a classic "sync said success but CJ shows
  nothing" cause; SPF's adapter handles this, worth knowing when reading
  sync logs).
- Per-feed CJ settings (program name, delimiter, remote filename) live in
  the feed's channel configuration; a connected channel with an unfinished
  per-feed config skips syncing rather than erroring.
- Some CJ setups run URL-pull instead of push; those regenerate with the
  normal feed pipeline and CJ fetches the URL — no SFTP involved.

## Sync etiquette (any channel)

- One sync at a time: if a sync is already in progress the tool refuses and
  names the in-flight sync log — poll `spf_channel_status` instead of
  retrying.
- A channel with no feeds assigned can't sync; assign a feed in the app
  first.
- Sync logs carry status (pending/success/failed), row counts, and error
  details — quote them rather than guessing at delivery state.

## Feed variants

- One catalog can ship as multiple feeds: a primary feed, language feeds
  (translated via the store's own translations), and market feeds (scoped
  selections with their own target market). Health, rules, mappings, and
  cells are all **per-feed** — always confirm which `feed_…` id you're
  operating on.
