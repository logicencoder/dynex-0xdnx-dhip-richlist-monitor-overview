# 0xDNX DHIP Richlist — monitor service

Long-running **Python indexer** for **Wrapped Dynex (0xDNX)** held in the **DHIP v2** pool on Ethereum. It watches DHIP `Transfer` events through a local Geth node, rebuilds holder balances and rankings, and pushes JSON to WordPress so [logicencoder.com/0xdnx-dhip-v2-richlist/](https://logicencoder.com/0xdnx-dhip-v2-richlist/) never scrapes Etherscan on every page view.

Pool share shifts whenever large wallets deposit or exit — a static export is stale within hours. This worker tails new blocks after an optional lookback catch-up, persists history in SQLite, and hands ranked data to the plugin that powers the public table, stats band, and live refresh loop.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Worker | Python 3, `web3.py`, `requests`, `orjson`, `sqlite3`, `configparser`, `threading` / `queue` |
| Chain access | Ethereum JSON-RPC (`eth_getLogs`, block heads); WebSocket subscription when available, HTTP poll fallback |
| Persistence | SQLite local database; JSON export `dhip_richlist_data.json`; optional `dhip_transaction.log` |
| WordPress handoff | Authenticated REST richlist push to the private plugin ingest endpoint |
| Configuration | `0xdnxdhip_config.ini`, `.env` fallbacks, extensive CLI flags for headless USM runs |
| Operations | Universal Service Manager long-running service on operator Linux host |
| Downstream UI | [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview) — shortcode, AJAX polls, crawler snapshots |

## Contracts indexed

| Name | Mainnet address |
|------|-----------------|
| DHIP v2 pool | `0xD76f48605DB0eBc75Ddc569340eaca42B7f65105` |
| Wrapped DNX (0xDNX) | `0x9928a8600d14ac22c0be1e8d58909834d7ceaf13` |

Token amounts use **nine decimal places** end-to-end in the monitor math and formatted payloads.

## Block scanning and classification

Two phases keep the richlist current without re-scanning the full chain on every restart.

**Lookback catch-up** runs at startup when `--lookback-blocks` is set (production uses a short window). The worker walks recent heights in chunks (default **2000** blocks per `eth_getLogs` query), replays DHIP transfers, and rebuilds holder balances before live tail begins.

**Live tail** subscribes to new block heads when a WebSocket endpoint is reachable; otherwise it polls HTTP. Each queued height flows through `process_block_number`, which classifies activity relative to the pool contract:

- **Deposit** — 0xDNX moving into DHIP (mint into the pool from the holder’s perspective on the public page)
- **Withdrawal** — 0xDNX leaving DHIP
- Internal transfers between pool participants update balances and ranks without double-counting supply

After each rebuild the worker recalculates **rank**, **balance**, and **percentage of total pool supply** for every holder address.

## Local analytics and audit trail

SQLite keeps the current leaderboard plus historical series — balance, rank, and percentage changes over time, classified deposits and withdrawals, daily snapshots for long-range charts, percentile bucket counts, and top-holder concentration trends. A **push audit log** records each WordPress handoff with HTTP outcome and payload size for operator review.

A parallel **`dhip_richlist_data.json`** file mirrors the ranked list for offline inspection, backups, and debugging without opening the database.

**Resume pointer** `last_processed_block.txt` lets the service continue from the last processed height after restart instead of replaying from genesis.

## WordPress multi-site push

When **auto-send** is enabled, each successful richlist rebuild enqueues a background POST to every configured WordPress target. Configuration supports multiple sites via repeated `[WordPress]` / `[WordPress2]` sections in INI, or repeated `-w` / `-k` / `-n` CLI triplets.

Each payload carries the timestamped holder array (address, balance, percentage, rank, formatted strings), aggregate totals, holder count, block number, and a slice of recent classified transactions pulled from SQLite.

The monitor sends **richlist JSON only**. Static HTML snapshots, Schema.org JSON-LD, and bot user-agent routing are generated **inside the WordPress plugin** after ingest.

PHP never opens an Ethereum RPC connection; all on-chain truth originates here.

## What the plugin adds (public UI)

Cross-link for visitor-facing behaviour — documented fully in the plugin overview, summarised here so readers know the split of responsibility:

- **From monitor data:** holder count, total 0xDNX in pool, ranked table, recent deposit/withdraw feed, concentration trends in historical tables.
- **Computed in WordPress:** whale count (≥1% share), small-holder band (<0.1%), 24h activity count, live AJAX refresh, load-more pagination, and SEO snapshot files for crawlers.

## Headless operation and CLI

Production runs non-interactively under Universal Service Manager with flags such as database enable, decimal precision, initial WordPress push, auto-send, lookback depth, and verbosity.

When started without flags on a TTY, an interactive wizard walks through logging, database filename, separate timestamped export files, decimal precision, Etherscan bootstrap toggle, and lookback depth.

Notable CLI knobs operators use in practice:

| Flag | Purpose |
|------|---------|
| `--lookback-blocks` | Blocks to scan on startup before live tail |
| `--db` / `--dbfile` | Enable SQLite and choose database path |
| `--send-initial` | Push current richlist to WordPress on startup (background thread) |
| `-a` / `--autosend` | POST after every rebuild |
| `-w` / `-k` / `-n` (repeatable) | WordPress URL, API key, and label per site |
| `--use-etherscan` | Optional Etherscan-assisted bootstrap when local node history is incomplete |
| `--separate-files` | Write timestamped JSON/CSV copies per update |
| `--log` | Append human-readable audit lines to `dhip_transaction.log` |

Optional **Etherscan** reconciliation helps when the local node is missing ancient logs; day-to-day production relies on the co-located Geth instance for low-latency `eth_getLogs`.

## Reliability notes

WordPress pushes run on a **background thread** so a slow HTTP response does not stall block processing. Failed sends are logged in the push audit trail for operator review.

API keys and RPC endpoints stay in local configuration files on the operator host — never in the public overview or GitHub tree.

Private code: [dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor) · public UI [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview)

Guide: [Wrapped Dynex richlist explained](https://logicencoder.com/wrapped-dynex-richlist-0xdnx-dvhip-v2/)

See [REPOS.md](REPOS.md).

---

## Feature examples (two per capability)

#### Lookback catch-up at startup
1. You set a short lookback window on USM start and confirm the worker replays recent DHIP transfers before live tail begins.
2. You restart after downtime and confirm the resume pointer continues from the last processed block instead of scanning from genesis.

#### Live block tail
1. You run with WebSocket heads enabled and watch new blocks enqueue and process without manual refresh.
2. WebSocket drops and HTTP poll fallback keeps the richlist advancing until the subscription recovers.

#### Deposit and withdrawal classification
1. A large wallet sends 0xDNX into DHIP — you see a DEPOSIT row, updated rank, and percentage in the next rebuild.
2. A whale withdraws — balance drops, rank shuffles, and zero-balance addresses drop from the in-memory map.

#### SQLite history and audit
1. You enable the database and review balance_history rows after a volatile hour — each classified move links to block height and tx hash.
2. You inspect the WordPress push audit table after a failed POST and retry from wp-admin once the API key is fixed.

#### WordPress multi-site push
1. You configure two `-w`/`-k` site triplets and confirm each rebuild POSTs ranked JSON to every target in sequence.
2. You enable auto-send and watch the public richlist page update within seconds of an on-chain deposit — without PHP calling Ethereum RPC.

#### JSON export mirror
1. You open `dhip_richlist_data.json` on the operator host and compare holder count with the live WordPress table during debugging.
2. You enable separate timestamped exports and keep a folder of JSON/CSV snapshots for monthly compliance review.

#### Rank and percentage change tracking
1. A mid-tier holder jumps five ranks after a competitor withdraws — rank_history records the movement for later charts.
2. Quiet block produces no percentage_history noise because unchanged shares skip insert.

#### Etherscan bootstrap (cold start)
1. Local node lacks ancient logs and you run with Etherscan bootstrap once to seed holder balances before live tail.
2. You reconcile bootstrap totals against on-chain supply and confirm drift is within your tolerance before enabling auto-send.

#### Headless USM operation
1. You launch with database, auto-send, and lookback flags only — no TTY wizard — and confirm USM logs show steady block progress.
2. You bump lookback after a long outage and let Phase 1 catch-up finish before trusting live percentages on the public page.

#### Transaction log file
1. You enable human-readable logging and tail DEPOSIT/WITHDRAWAL lines during a known whale move for a support ticket.
2. You correlate a log line with Etherscan while the SQLite ledger provides the structured query path for the plugin feed.

#### Daily snapshots and concentration
1. Midnight UTC passes and daily_snapshots capture one row per holder for long-range charts in the plugin.
2. You compare top-10 concentration between two saves the same day and read the updated historical_totals row for that date.

#### Interactive wizard (dev)
1. On a laptop TTY you walk the wizard — logging, database path, separate files, lookback — before your first push to staging WordPress.
2. You disable auto-send in the wizard run, validate JSON locally, then enable `-a` for production once ranks look sane.


---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
