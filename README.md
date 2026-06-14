# useful-scripts

Personal **Logic Encoder** script toolbox — exchange monitors, MEXC protobuf helpers, and small utilities collected while building trading and monitoring stacks. Nothing here is a shipped product or WordPress plugin; these are scripts you run locally when debugging depth feeds, testing API keys, or fixing protobuf imports before wiring logic into larger apps.

The private code lives in [logicencoder/useful-scripts](https://github.com/logicencoder/useful-scripts). This public repo is a readable map of what exists and how the pieces fit together — no API keys, no hostnames.

## The problem it solves

Over years of MEXC and Gate.io work, the same prototypes kept getting copied under slightly different names — five `proto/` folders, three “final” balance monitors, dashboards that only lived on one machine. The 2026 cleanup gave every script a clear prefix (`mexc_*`, `gate_*`, `util_*`) and **one** protobuf pipeline so monitors stop breaking each other.

## MEXC protobuf pipeline

MEXC V3 WebSocket traffic is binary protobuf. Every monitor that reads depth or trades depends on the same two folders:

| Folder | Role |
|--------|------|
| `mexc_api_schema_from_github/` | Message definitions from [mexcdevelop/websocket-proto](https://github.com/mexcdevelop/websocket-proto) |
| `generated_proto/` | Compiled `*_pb2.py` decoders — **fixed name**; do not rename |

Run `mexc_proto_auto_setup.py` to download fresh schemas and recompile. Import through `mexc_proto_loader.py` so each script uses the same path. Use `mexc_proto_inspector.py` when imports fail after an API update.

## MEXC and Gate.io market data

**Dual exchange orderbook dashboard** (`mexc_gate_dual_orderbook_dashboard.py`, port **8017**) — FastAPI page with MEXC and Gate.io 20-level depth side by side plus an arbitrage summary panel. Handy when you want the same symbol on two CEXes without switching tabs.

**MEXC depth in the terminal** — Textual TUIs for live orderbooks: `mexc_protodepth_tui_single.py` for one symbol, `mexc_protodepth_tui_multi.py` for several. Lower latency than a browser when tuning protobuf decode.

**MEXC depth in the browser** — `mexc_protodepth_web_dashboard.py` serves a lightweight dashboard on port **8010** for local sharing during development.

**Gate orderbook terminal** — `gate_orderbook_terminal.py` streams Gate.io WebSocket depth to the terminal with USD totals; Gate-only debugging without the MEXC protobuf stack.

**Trades and combined feeds** — `mexc_trades_fetcher_v3.py`, `gate_trades_fetcher.py`, and `mexc_ws_depth_and_trades.py` log trades, validate channels, and prototype asyncio loops before they land in production apps.

## MEXC balance and orders

Standalone testers around private user-data WebSocket and REST flows:

- `mexc_balance_monitor_v3.py` and `mexc_balance_monitor_v3_textual.py` — balance views (plain and Textual UI)
- `mexc_listen_key_checker.py` — listen-key lifecycle checks
- `mexc_ensure_single_buy_order.py` — keep a single buy order open
- `mexc_single_immediate_buy.py` — fire an immediate buy
- `mexc_dynamic_avgbuy.py` — dynamic average-buy math

Copy `config/mexc_keys.example.json` to `config/mexc_keys.json` for scripts that need MEXC API access. Keys stay local and gitignored.

## Ethereum, DEX, and ops helpers

One-off scripts around the same machines that run trading stacks: `eth_gas_price_checker_ws.py` (local Geth gas over WebSocket), `dex_liquidity_pool_monitor.py`, `open_firewall_port.sh`, `samba_setup.sh`, `tmux_mexc_dnx_logger.sh`, `kill_mexc_ticker_tunnel_processes.sh`, and similar small ops tools.

## Generic utilities (`util_*`)

Reusable snippets — CLI input helpers, Flask client info, token decimals, colored logs, terminal title, Textual button demo, graceful shutdown, site indexing check, MP4→GIF, network card speed. Kept here so they do not multiply across repos.

## Discord experiments

The `discord/` folder holds two Node scripts for discord.js reply-grouping experiments — not production bots.

## Quick start

```bash
git clone  # private repo — see REPOS.md
cd useful_scripts

python3 mexc_proto_auto_setup.py          # refresh GitHub schemas + generated_proto
python3 mexc_gate_dual_orderbook_dashboard.py
```

## Related Logic Encoder repos

| Repo | How it differs |
|------|----------------|
| [cex_dex_arb_app](https://github.com/logicencoder/cex_dex_arb_app) | Full CEX/DEX arb product with its own protobuf copy |
| [mexc_trading_app](https://github.com/logicencoder/mexc_trading_app) | Production trading stack |
| **useful-scripts** | Personal toolbox — prototypes, monitors, protobuf fixes |

The same protobuf *idea* appears in multiple repos; **useful-scripts** is the cleaned canonical copy for standalone scripts only.

## Changelog (high level)

| Date | Change |
|------|--------|
| 2026-06 | Renamed scripts, merged duplicate protos, single `generated_proto/`, GitHub publish |

See [REPOS.md](REPOS.md) for the public/private repo pair.

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
