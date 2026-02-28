# SwapDesk SKILL.md

This file extends the upstream IntercomSwap `SKILL.md` with instructions specific to the **SwapDesk** UI layer.

Agents using this fork should treat this file as the canonical guide for installing, running, and interacting with SwapDesk on top of IntercomSwap.

---

## What SwapDesk Adds

SwapDesk is a single-file conversational frontend (`ui/swapdesk/swapdesk.html`) that sits on top of the full IntercomSwap stack. It provides:

- Natural language chat interface for BTC ⚡ ↔ USDT swaps
- Live BTC/USDT price from CoinGecko API (refreshed every 30s)
- Live 24H volume and market cap from CoinGecko (refreshed every 60s)
- Live BTCUSDT trade feed from Binance public API (refreshed every 10s)
- Real-time swap quotes with accurate fee breakdown (0.1% platform + 0.1% trade fee)
- Step-by-step swap flow tracker (Quote → LN Pay → Escrow → Claim)
- Quick BTC ↔ USDT calculator using live price
- Automatic peer detection via SC-Bridge WebSocket on `ws://127.0.0.1:49222`
- Live mode when peer is connected, Demo mode when not — clearly labeled either way

---

## Requirements

Same as upstream IntercomSwap:

- Node.js 18+
- Pear runtime
- A running Lightning node (CLN or LND)
- A Solana wallet with USDT (for maker role)

Plus for SwapDesk UI:

- Any modern web browser (Chrome, Firefox, Brave)

---

## Installation

### Step 1 — Clone this fork

```bash
git clone https://github.com/YOUR_GITHUB_USERNAME/intercom-swap.git
cd intercom-swap
```

### Step 2 — Install dependencies

```bash
npm install
```

### Step 3 — Run tests

```bash
npm test
npm run test:e2e
```

### Step 4 — Start your IntercomSwap peer

For testing (local regtest):

```bash
scripts/run-swap-maker.sh swap-maker 49222 0000intercomswapbtcusdt
```

For mainnet:

```bash
scripts/run-swap-maker.sh swap-maker-mainnet 49222 0000intercomswapbtcusdt \
  --ln-network mainnet \
  --solana-rpc-url https://api.mainnet-beta.solana.com
```

> Important: always use port `49222` as the SC-Bridge port so SwapDesk can detect it automatically.

### Step 5 — Open SwapDesk

```
ui/swapdesk/swapdesk.html
```

Open directly in your browser. No build step, no server required.

SwapDesk will attempt to connect to `ws://127.0.0.1:49222` on load and every 15 seconds. When the peer is running it switches automatically from Demo Mode to Live Mode.

---

## Peer Detection Behavior

SwapDesk tries to open a WebSocket connection to `ws://127.0.0.1:49222` with a 2-second timeout:

| Result | UI State |
|---|---|
| Connection succeeds | Header badge: ✓ PEER ONLINE (green) · Swap button active |
| Connection fails or times out | Header badge: DEMO MODE · Swap button shows instructions |

Re-check runs automatically every 15 seconds — no page reload needed.

---

## How Agents Should Use SwapDesk

Agents interact with the underlying IntercomSwap peer via the SC-Bridge (same as upstream). SwapDesk is the human-facing layer on top.

SwapDesk reads from:
- CoinGecko public API for BTC market data
- Binance public API for live trade feed
- SC-Bridge WebSocket at `ws://127.0.0.1:<scPort>` for peer interaction

### Example agent prompts

```
Start the IntercomSwap peer on port 49222, then open SwapDesk and confirm PEER ONLINE status.
```

```
Use SwapDesk to get a live swap quote for 0.01 BTC.
```

```
Open SwapDesk and post an RFQ for 0.005 BTC on the 0000intercomswapbtcusdt channel.
```

```
Check the current BTC/USDT rate shown in SwapDesk and compare it to the live CoinGecko feed.
```

---

## Triggering Swaps via SwapDesk

When the peer is online, the swap button calls `initiateSwap()` which posts an RFQ to the `0000intercomswapbtcusdt` sidechannel via the SC-Bridge WebSocket.

| User Input | SwapDesk Action |
|---|---|
| "Swap 0.001 BTC for USDT" | Calculates live quote, shows swap card with Initiate button |
| "Sell 500 USDT for BTC" | Calculates reverse quote, shows swap card |
| "What's the rate?" | Fetches and displays live CoinGecko data |
| "Show fees" | Displays IntercomSwap fee structure from protocol |
| "Check market volume" | Displays live CoinGecko 24H stats |
| "Is my peer connected?" | Reports live peer detection status |

---

## File Location

```
ui/swapdesk/swapdesk.html
```

Self-contained single HTML file. No build tools, no npm install, no external server. Open in any browser.

---

## Trac Address

> **trac1waavv98ys2seavfcv0vnax5z5ke4kvncw8ay8h88x7g5y2a37pesy5dwz8**

---

## Upstream

This fork builds on:

- IntercomSwap: https://github.com/TracSystems/intercom-swap
- Intercom: https://github.com/Trac-Systems/intercom

For core swap operations (LN, Solana, RFQ bots, recovery), refer to the upstream `SKILL.md` and `README.md`.
