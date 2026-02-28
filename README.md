# SwapDesk — BTC ⚡ USDT Swap Assistant

> A human-friendly conversational UI built on top of [IntercomSwap](https://github.com/TracSystems/intercom-swap) — the peer-to-peer Bitcoin Lightning ↔ USDT Solana swap protocol.

---

## What is SwapDesk?

**SwapDesk** is a frontend application that makes IntercomSwap accessible to everyone — not just developers.

Instead of running terminal commands and reading JSON envelopes, users simply type what they want in plain English:

> *"Swap 0.001 BTC for USDT"*
> *"What's the current rate?"*
> *"Show me the fees"*

SwapDesk handles the rest — fetching live market data, displaying real-time quotes with fee breakdowns, showing step-by-step swap progress, and connecting directly to a running IntercomSwap peer via the SC-Bridge WebSocket when available.

---

## Features

- **Conversational swap interface** — natural language input, no terminal required
- **Live BTC/USDT price** — real-time from CoinGecko API, refreshed every 30 seconds
- **Live 24H market stats** — volume (BTC + USD) and market cap from CoinGecko, refreshed every 60 seconds
- **Live trade feed** — real BTCUSDT trades from Binance public API, refreshed every 10 seconds
- **Real-time swap quotes** — calculated against live price with accurate fee breakdown (0.1% platform + 0.1% trade)
- **Step tracker** — visual progress through Quote → LN Pay → Escrow → Claim
- **Quick BTC ↔ USDT calculator** — converts in real-time using live price
- **Peer detection** — automatically detects a running IntercomSwap peer on `ws://127.0.0.1:49222`, re-checks every 15 seconds
- **Live mode** — when peer is online, swap button posts a real RFQ to the `0000intercomswapbtcusdt` sidechannel
- **Demo mode** — when no peer is running, all market data is still real but swap execution is clearly labeled as unavailable
- **Non-custodial** — built on HTLC settlement, no funds ever touch SwapDesk

---

## Live vs Demo Mode

SwapDesk automatically detects whether a local IntercomSwap peer is running:

| State | What happens |
|---|---|
| **Peer online** (port 49222) | Badge shows ✓ PEER ONLINE · Swap button posts a real RFQ to the network |
| **No peer** | Badge shows DEMO MODE · Market data is still real · Swap execution disabled with clear explanation |

---

## Architecture

```
User (browser)
      |
      | natural language input
      v
SwapDesk UI (ui/swapdesk/swapdesk.html)
      |
      +-- CoinGecko API ──────── live BTC price, 24H stats
      |
      +-- Binance public API ─── live BTCUSDT trade feed
      |
      +-- SC-Bridge WebSocket ── ws://127.0.0.1:49222 (if peer running)
            |
            v
      IntercomSwap Peer (this fork)
            |
            +── P2P Sidechannels (RFQ negotiation)
            +── Lightning Network (BTC leg)
            +── Solana (USDT escrow + HTLC settlement)
```

---

## Quick Start

### 1. Install dependencies

```bash
npm install
```

### 2. Start your IntercomSwap peer

```bash
scripts/run-swap-maker.sh swap-maker 49222 0000intercomswapbtcusdt
```

> Port `49222` is the SC-Bridge port SwapDesk listens for. Keep it consistent.

### 3. Open SwapDesk

```
ui/swapdesk/swapdesk.html
```

Open this file in your browser. SwapDesk will detect the running peer within 15 seconds and switch from Demo Mode to Live Mode automatically.

No build step. No server. Just open and use.

---

## Data Sources

| Data | Source | Refresh |
|---|---|---|
| BTC/USDT price + 24h change | CoinGecko API | Every 30s |
| 24H volume + market cap | CoinGecko API | Every 60s |
| Live trade feed | Binance public API | Every 10s |
| Swap fees (0.1% + 0.1%) | IntercomSwap protocol | Fixed |
| Peer connection | SC-Bridge WebSocket | Every 15s |

---

## Repo Structure

```
/
├── ui/
│   └── swapdesk/
│       └── swapdesk.html     ← SwapDesk frontend (this app)
├── src/                      ← IntercomSwap core (upstream)
├── scripts/                  ← IntercomSwap scripts (upstream)
├── contract/                 ← Solana escrow program (upstream)
├── SKILL.md                  ← Agent instructions (updated for SwapDesk)
└── README.md                 ← This file
```

---

## Built On

This repo is a fork of **[IntercomSwap](https://github.com/TracSystems/intercom-swap)** which is itself a fork of **[Intercom](https://github.com/Trac-Systems/intercom)** by Trac Systems.

SwapDesk adds a human-friendly UI layer on top of the full IntercomSwap stack without modifying any core swap logic.

---

## Trac Address

> **trac1waavv98ys2seavfcv0vnax5z5ke4kvncw8ay8h88x7g5y2a37pesy5dwz8**

---

## License

MIT — same as upstream IntercomSwap.
