# 0xDNX DHIP Richlist Monitor — overview

Public description of the **private Python monitor** that indexes Wrapped Dynex (0xDNX) in **DHIP v2** on Ethereum and feeds the LogicEncoder richlist. Private code: [dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor).

**Live site (WordPress):** https://logicencoder.com/0xdnx-dhip-v2-richlist/  
**Plugin overview:** [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview)

## What it is

**What:** Long-running monitor on operator infrastructure that tracks holder balances, concentration metrics, and recent deposits/withdrawals for 0xDNX in DHIP v2, then POSTs payloads to the WordPress plugin.  
**Why:** On-chain truth must be computed once near the node/RPC; the public site should not scrape Etherscan per page view.  
**Who:** Token holders and researchers see rankings on logicencoder.com; operator runs the monitor on SOL.

## Full stack

```
Ethereum (0xDNX in DHIP v2)
        │
        ▼
Python monitor (private — this overview)
        │  POST richlist + snapshot payloads
        ▼
WordPress plugin (private PHP)
        │
        ├── Visitors — live AJAX table
        └── Crawlers — static snapshot HTML + JSON-LD
```

| Repo | Role |
|------|------|
| **This overview** | Monitor capabilities (no secrets) |
| [dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor) | Private indexer |
| [dynex-0xdnx-dhip-richlist-plugin](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin) | Private WP plugin |
| [dynex-0xdnx-dhip-richlist-plugin-overview](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-plugin-overview) | Public plugin UX/docs |

## Metrics the monitor drives

**What:** Total holders, supply in pool, top-10/top-50 concentration, whale counts, 24h deposit/withdraw activity, ranked holder table with percentages.  
**Why:** Investors and the community need transparency on distribution and centralization risk.  
**Who:** Visitors on the public richlist page; SEO/AI crawlers consume plugin-generated snapshots.

## Live transaction feed

**What:** Classified deposit/withdraw events with amounts, timestamps, and explorer links.  
**Why:** Rankings alone hide flow; activity shows whether whales are entering or exiting the program.  
**Who:** Researchers monitoring program health.

## Production

**What:** Runs on SOL under `/home/sol/lojzo/0xdnxdhip_richlist_monitor/` (see private `ARCHITECTURE.md`).  
**Why:** Co-locate with other LogicEncoder chain tools and stable outbound POST to Hostinger.  
**Who:** Operator maintains RPC keys and monitor uptime.

## Related documentation

Guide: https://logicencoder.com/wrapped-dynex-richlist-0xdnx-dvhip-v2/

See [REPOS.md](REPOS.md).

## Licensing

© LogicEncoder.
