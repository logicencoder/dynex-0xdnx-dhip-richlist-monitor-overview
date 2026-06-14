# 0xDNX DHIP Richlist Monitor

Private Python indexer for **Wrapped Dynex (0xDNX)** in **DHIP v2** on Ethereum. Scans chain activity, maintains holder balances and concentration metrics, and pushes payloads to the WordPress richlist — so the public site does not scrape Etherscan on every page view.

Private source: [logicencoder/dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor).

**Public richlist (visitors):** [logicencoder.com/0xdnx-dhip-v2-richlist/](https://logicencoder.com/0xdnx-dhip-v2-richlist/)  
**Plugin overview (pair, later):** [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview)

## The problem it solves

Holder distribution, whale concentration, and recent deposit/withdraw activity need a **continuous indexer** near Ethereum RPC — not per-request explorer queries. This monitor builds the dataset the public richlist table and SEO snapshots consume.

## Metrics produced

- Total unique holders and supply locked in DHIP v2
- Top-10 and top-50 concentration percentages
- Whale counts and 24h deposit/withdraw activity
- Ranked holder table with exact % of supply
- Live transaction feed with deposit/withdraw classification and Etherscan links

## Runtime behavior

Batch block processing with SQLite persistence, balance updates from classified transactions, richlist JSON generation, and periodic push to the WordPress plugin endpoint. Lookback catch-up after downtime, then tail new blocks.

## Stack

Python 3, Web3 HTTP provider, SQLite (`0xdnxdhip_holders.db`), JSON richlist artifacts. Full schema and CLI flags in the private `ARCHITECTURE.md`.

See [REPOS.md](REPOS.md) for repository links.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
