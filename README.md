# 0xDNX DHIP Richlist — monitor service

Private Python indexer for **Wrapped Dynex (0xDNX)** in **DHIP v2** on Ethereum. Scans `Transfer` events on the DHIP contract, maintains holder balances and concentration history in SQLite, builds richlist JSON, and POSTs payloads to the WordPress plugin — so the public page never queries Etherscan per view.

**Private code:** [logicencoder/dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor)

**Public product (portfolio):** [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview) — [logicencoder.com/0xdnx-dhip-v2-richlist/](https://logicencoder.com/0xdnx-dhip-v2-richlist/)

The plugin renders the ranked table, stats band, recent transactions, and bot snapshots. This monitor is the on-chain indexer.

## Contracts tracked

| Token / pool | Address (mainnet) |
|--------------|-------------------|
| DHIP v2 pool | `0xD76f48605DB0eBc75Ddc569340eaca42B7f65105` |
| Wrapped DNX (0xDNX) | `0x9928a8600d14ac22c0be1e8d58909834d7ceaf13` |

Decimals: **9**. RPC via configurable HTTP Web3 provider (and optional WebSocket for live heads).

## Execution modes

**Lookback catch-up** — scans a configurable block range in chunks (default chunk size **2000**) using `eth_getLogs` on the DHIP contract, rebuilds holder balances, writes SQLite and JSON, optionally pushes to WordPress.

**Live tail** — subscribes to new block heads (WebSocket or HTTP poll), queues each height through `process_block_number`, classifies mint/burn/transfer, updates ranks and percentages.

## Classification and metrics

Each transfer updates holder balances and aggregate stats:

- Total unique holders and supply in the pool
- Top-10 / top-50 / top-100 concentration over time
- Percentile distribution buckets and daily snapshots
- 24h deposit and withdraw counts for the activity band
- Ranked table with exact **% of total supply** per address

Recent transactions feed classifies **deposit vs withdraw** relative to the pool for the plugin’s live AJAX UI.

## Persistence

SQLite **`0xdnxdhip_holders.db`** tables include `holders`, `balance_history`, `transactions`, `stats`, `percentage_history`, `rank_history`, `daily_snapshots`, `percentage_distribution`, `historical_totals`, and `wordpress_send_log` for push audit.

JSON artifact **`dhip_richlist_data.json`** mirrors the ranked list for offline inspection and plugin ingest.

## WordPress integration

Authenticated **`POST /wp-json/0xdnxdhip/v1/richlist`** (and related routes) from the monitor after each successful rebuild. Plugin stores rows, serves **`[0xdnxdhip_richlist]`** shortcode, generates static snapshot HTML for crawlers, and exposes public **`GET /stats`** for AJAX refresh.

PHP never calls Ethereum — all chain truth originates here.

## Key file

**`0xdnxdhip_richlist_monitor.py`** — CLI flags for lookback depth, chunk size, push targets, and logging. Full schema and env vars in private `ARCHITECTURE.md`.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
