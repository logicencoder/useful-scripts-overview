# useful-scripts — overview

Personal Logic Encoder script toolbox: exchange monitors, protobuf tooling, and small utilities accumulated over years of crypto/dev work. Reorganized in 2026 so names and folders finally make sense.

**Code repo (private):** [logicencoder/useful-scripts](https://github.com/logicencoder/useful-scripts)  
**This repo:** public map of what exists and why — no API keys, no internal hosts.

https://logicencoder.com

---

## What this is

Not a product, not a framework, not something USM deploys. It is an operator’s **working drawer** of Python/bash/JS scripts that grew while building Logic Encoder trading and monitoring stacks. Think: “scripts I actually run when debugging MEXC depth, Gate orderbooks, or fixing protobuf on a laptop or SOL.”

After cleanup, one rule applies everywhere:

- **GitHub API schemas** live in `mexc_api_schema_from_github/` (downloaded from MEXC’s official repo)
- **Compiled decoders** live in `generated_proto/` (fixed folder name — monitors import from here)
- **One loader** — `mexc_proto_loader.py` — so every monitor uses the same import path

---

## Who this is for

| Audience | Why read this |
|----------|----------------|
| **Future me** | Remember which script is the “real” one after six renamed copies |
| **Logic Encoder ops** | Know what runs manually on WSL vs what was synced to SOL |
| **Collaborators** | See scope before asking for a polished npm package |

---

## MEXC + Gate.io — market data

### Dual exchange orderbook dashboard

**What:** FastAPI page showing MEXC and Gate.io 20-level depth side by side, with an arbitrage summary panel.  
**Why:** Compare the same symbol on two CEXes without switching browser tabs.  
**Script:** `mexc_gate_dual_orderbook_dashboard.py` — default port **8017**.

### MEXC depth (terminal UIs)

**What:** Textual terminal dashboards for live orderbook. Single-coin and multi-coin variants.  
**Why:** Lower latency than a browser when tuning symbols or testing protobuf decode.  
**Scripts:** `mexc_protodepth_tui_single.py`, `mexc_protodepth_tui_multi.py`.

### MEXC depth (web)

**What:** Lightweight FastAPI dashboard for one symbol.  
**Why:** Shareable local URL during development. Port **8010**.

### Gate orderbook (terminal)

**What:** Gate.io WebSocket depth printed to the terminal with USD totals and multi-connection support for many pairs.  
**Why:** Gate-only debugging without the MEXC protobuf stack.  
**Script:** `gate_orderbook_terminal.py`.

### Trades and combined depth+trades

**What:** Fetchers and combined asyncio tools for MEXC trades, Gate trades, and MEXC depth+trades in one loop.  
**Why:** Log to files, validate WS channels, prototype before wiring into larger apps.  
**Scripts:** `mexc_trades_fetcher_v3.py`, `gate_trades_fetcher.py`, `mexc_ws_depth_and_trades.py`.

---

## MEXC — balance and orders

**What:** Balance monitors (plain and Textual UI), listen-key checker, helpers to keep a single buy order or fire an immediate buy, dynamic average-buy math.  
**Why:** Private MEXC user-data WebSocket and REST need quick standalone testers outside `mexc_trading_app`.  
**Scripts:** `mexc_balance_monitor_v3.py`, `mexc_balance_monitor_v3_textual.py`, `mexc_listen_key_checker.py`, `mexc_ensure_single_buy_order.py`, `mexc_single_immediate_buy.py`, `mexc_dynamic_avgbuy.py`.

---

## MEXC API files from GitHub

**What:** Two folders — schemas downloaded from GitHub, Python modules compiled from them.  
**Why:** MEXC V3 WebSocket is protobuf; you need one canonical copy instead of five scattered `proto/` directories (the old “bordelik”).  

| Folder | Plain English |
|--------|----------------|
| `mexc_api_schema_from_github/` | Official message definitions from [mexcdevelop/websocket-proto](https://github.com/mexcdevelop/websocket-proto) |
| `generated_proto/` | Compiled `*_pb2.py` — **do not rename**; all monitors depend on it |

**Setup:** `mexc_proto_auto_setup.py` downloads and compiles. **Inspect:** `mexc_proto_inspector.py`. **Load in code:** `mexc_proto_loader.py`.

---

## Ethereum, DEX, infra helpers

**What:** Local Geth gas WS checker, DEX liquidity pool monitor, firewall port script, Samba setup, tmux logger, process killer for stale tunnel scripts, Windows audio restart bat.  
**Why:** One-off ops tasks around the same machines that run trading stacks.  
**Examples:** `eth_gas_price_checker_ws.py`, `dex_liquidity_pool_monitor.py`, `open_firewall_port.sh`.

---

## Generic utilities (`util_*`)

**What:** CLI input, Flask client info sender, token decimals, colored logs, terminal title, Textual button demo, graceful shutdown, site indexing check, MP4→GIF, network card speed.  
**Why:** Copy-paste starting points for other projects — kept here so they do not multiply across repos.

---

## Discord experiments

**What:** Two Node scripts for discord.js reply grouping experiments.  
**Folder:** `discord/` — not production Discord bots.

---

## Archive (`!old/`)

**What:** Superseded filenames (`protodepth_20_1a`, `bal_mon_mexc_*`, `mexc_depth_*`, August 2025 backup snapshot).  
**Why:** Git history of the bordelik — reference only, not daily drivers. Mapping in private repo `!old/README.md`.

---

## How it is run

```bash
cd ~/useful_scripts   # WSL
# or /home/sol/lojzo/useful_scripts on SOL

python3 mexc_proto_auto_setup.py          # refresh GitHub schemas + generated_proto
python3 mexc_gate_dual_orderbook_dashboard.py
```

Copy `config/mexc_keys.example.json` → `config/mexc_keys.json` for scripts that need MEXC API access. Keys stay local and gitignored.

---

## Relationship to other Logic Encoder repos

| Repo | Difference |
|------|------------|
| `cex_dex_arb_app` | Full CEX/DEX arb product with its own `generated_proto/` |
| `mexc_trading_app` | Production trading stack on SOL |
| **useful-scripts** | Personal toolbox — prototypes, monitors, protobuf fixes |

Same protobuf *idea* appears in multiple repos; **useful-scripts** is the cleaned canonical copy for standalone scripts only.

---

## Changelog (high level)

| Date | Change |
|------|--------|
| 2026-06 | Renamed scripts (`mexc_*`, `gate_*`, `util_*`), merged duplicate protos, `mexc_api_schema_from_github/` + single `generated_proto/`, synced WSL↔SOL, first GitHub publish |

---

Logic Encoder — https://logicencoder.com
