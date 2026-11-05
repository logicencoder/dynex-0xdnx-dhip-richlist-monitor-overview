# 0xDNX DHIP Richlist — Holder Monitoring

Public description of the **private Python monitor** that indexes Wrapped Dynex (0xDNX) in **DHIP v2** on Ethereum and feeds the Logic Encoder richlist. Private code: [dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor).

A pool's holder distribution changes as participants deposit, withdraw and transfer their holdings. The 0xDNX DHIP Richlist monitor keeps that changing picture available for research: who holds a share, how the ranking has moved and which recent activity deserves attention.

It supplies the public [DHIP v2 richlist](https://logicencoder.com/0xdnx-dhip-v2-richlist/) and its connected WordPress dashboard. Readers can explore the distribution on the website, while the operator retains history and tools for reviewing the information behind it.

## Tech stack

| Layer | Technologies |
|---|---|
| Application | Python |
| Chain and analysis | Ethereum, Web3.py |
| History | SQLite |
| Publishing | WordPress, JSON |

## Follow the changing pool

The monitor follows Wrapped Dynex holdings in the DHIP v2 pool and classifies every movement as a deposit, a withdrawal or a transfer between participants, so a movement keeps its pool context instead of being treated as generic activity. It runs in two modes. A **look-back catch-up** scans a chosen number of recent blocks in chunks, which rebuilds the picture after a pause. A **live tail** then follows new blocks and keeps the picture current.

A useful review begins with a recent event, then checks the affected holder's balance and place in the distribution. The information supports investigation without assigning a motive to the participant.

## Compare balances, ranks and shares

A ranked holder view brings balance, rank and percentage of pool holdings together. These answer different questions: the amount held, the position relative to other participants and the share of the tracked total.

The ranking can be reviewed as a whole or used to follow a specific address. Changes in one participant can then be understood alongside changes elsewhere in the pool.

## Return to earlier observations

The monitor stores every processed holder balance and event in a local history, so the picture is not limited to the present moment. Compare the distribution across selected observations, revisit a period of activity, or inspect how the position and share of a tracked holder changed over time. Recent deposits and withdrawals provide another route into the same history: start from the movement, then follow the holder it affected.

Because the history is rebuilt from chain logs, it can be extended backwards at any time by running a catch-up over an earlier block range.

## Keep public pages informed

The operator can publish the tracked information to more than one configured WordPress site. Automatic updates support an ongoing richlist, while publication records help identify a site that needs attention.

The companion WordPress product provides the visitor-facing table, summaries and transaction browsing. Its purpose is to make the holder information accessible within the website; the monitor supplies the chain observations behind that experience.

## Scope and limits

The holder picture is rebuilt from public Ethereum logs of the DHIP v2 contract, with balances in the token's nine-decimal units and shares expressed against the total supply of the programme. After every batch the totals are reconciled, and an optional on-chain check compares the aggregate with the contract. Numbers are as accurate as the chain data and the contract's events; the monitor reports observations, not intentions, and does not label holders.

---

Private code: [dynex-0xdnx-dhip-richlist-monitor](https://github.com/logicencoder/dynex-0xdnx-dhip-richlist-monitor)

Logic Encoder · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
