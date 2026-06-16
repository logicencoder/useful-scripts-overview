# useful-scripts

Personal **Logic Encoder** script toolbox — MEXC and Gate.io market-data prototypes, protobuf setup helpers, balance and order testers, chain gas checks, and small `util_*` snippets collected while building trading and monitoring stacks. These are **local scripts** you run from a checkout when debugging depth feeds, validating API keys, or fixing protobuf imports before wiring logic into production apps — not a shipped product, WordPress plugin, or fleet dashboard.

The private tree lives in [logicencoder/useful-scripts](https://github.com/logicencoder/useful-scripts). This public repo is a readable map of what exists and when to reach for each script — no API keys, hostnames, or operator paths.

After the 2026 cleanup every script uses a clear prefix (`mexc_*`, `gate_*`, `util_*`) and **one shared protobuf pipeline** so monitors stop breaking each other when MEXC updates WebSocket schemas on GitHub.

- **Single `generated_proto/` decoder tree** — all MEXC monitors import through `mexc_proto_loader.py`.
- **Refresh from upstream** — `mexc_proto_auto_setup.py` downloads schemas and recompiles after API changes.
- **Terminal and browser depth tools** — Textual TUIs, FastAPI dashboards, and plain loggers for the same binary feeds production apps use.
- **Order and balance sandboxes** — isolated scripts to test listen keys, single-order guards, and immediate buys without the full trading stack.
- **Ops one-liners** — firewall, Samba, tmux loggers, tunnel cleanup, protobuf version fixes.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Languages | Python 3, Bash, small Node snippets in `discord/` |
| MEXC feeds | Spot V3 protobuf WebSocket, REST where scripts need bootstrap |
| Gate feeds | WebSocket depth and trades (no MEXC protobuf stack) |
| UI | Textual + Rich TUIs, FastAPI + uvicorn for local dashboards |
| Protobuf | `mexc_api_schema_from_github/` (`.proto` from [mexcdevelop/websocket-proto](https://github.com/mexcdevelop/websocket-proto)), `generated_proto/` (`*_pb2.py`) |
| Chain | web3.py, local Geth WebSocket gas checker, Uniswap V3 pool liquidity poll |
| Config | `config/mexc_keys.json` from `mexc_keys.example.json` — gitignored, local only |
| Hosting | Manual runs from a developer checkout on WSL or Linux |

## MEXC protobuf pipeline

MEXC V3 private and public WebSocket traffic is **binary protobuf**. Depth monitors, trade loggers, and balance tools all depend on the same two folders:

| Folder | Role |
|--------|------|
| `mexc_api_schema_from_github/` | Message definitions synced from MEXC’s GitHub proto repo |
| `generated_proto/` | Compiled `*_pb2.py` decoders — **fixed folder name**; scripts import from here |

**`mexc_proto_auto_setup.py`** — downloads fresh `.proto` files, runs `protoc`, and repopulates `generated_proto/`. Run after MEXC publishes schema changes or when imports fail on a new machine.

**`mexc_proto_loader.py`** — single import path every `mexc_*` monitor uses so paths stay consistent.

**`mexc_proto_inspector.py`** — prints folder health, import errors, and version mismatches when debugging a broken decode.

**`protobuf_vscode_import_fix.py`** and **`protobuf_downgrade_to_3_20.sh`** — editor import paths and pip protobuf version pinning when `protoc` and runtime disagree.

## MEXC and Gate.io market data

**Dual exchange orderbook dashboard** (`mexc_gate_dual_orderbook_dashboard.py`) — FastAPI page on port **8017** with MEXC and Gate.io 20-level depth side by side plus a spread summary panel. Use when you want the same symbol on two CEXes in one browser tab during research.

**MEXC depth in the terminal** — `mexc_protodepth_tui_single.py` for one symbol, `mexc_protodepth_tui_multi.py` for several. Textual layouts show bids/asks, cumulative size, weighted mid, and connection debug — lower friction than a browser when tuning protobuf decode latency.

**MEXC depth in the browser** — `mexc_protodepth_web_dashboard.py` on port **8010** for sharing a live book during pair debugging.

**Gate orderbook terminal** — `gate_orderbook_terminal.py` streams Gate WebSocket depth with USD totals; Gate-only work without loading the MEXC protobuf stack.

**Trades and combined feeds** — `mexc_trades_fetcher_v3.py` and `gate_trades_fetcher.py` log public trades to rotating files; `mexc_ws_depth_and_trades.py` prototypes combined depth+trade asyncio loops before they move into larger apps.

## MEXC balance and orders

Standalone testers around private user-data WebSocket and REST flows:

| Script | Use when |
|--------|----------|
| `mexc_balance_monitor_v3.py` | Plain terminal balance polling |
| `mexc_balance_monitor_v3_textual.py` | Same data in a Textual panel layout |
| `mexc_listen_key_checker.py` | Validate listen-key create, extend, and expiry behaviour |
| `mexc_ensure_single_buy_order.py` | Keep exactly one buy order open on a symbol |
| `mexc_single_immediate_buy.py` | Fire a one-shot market/limit buy for API smoke tests |
| `mexc_dynamic_avgbuy.py` | Experiment with dynamic average-buy sizing math |

Copy `config/mexc_keys.example.json` to `config/mexc_keys.json` for scripts that need MEXC credentials. Keys never ship in git.

## Ethereum, DEX, and ops helpers

| Script | Use when |
|--------|----------|
| `eth_gas_price_checker_ws.py` | Poll gas and block stats from a **local Geth** WebSocket with coloured terminal output |
| `dex_liquidity_pool_monitor.py` | Watch Uniswap V3 pool liquidity for a configured token/WETH pair |
| `open_firewall_port.sh` | Open a UFW port during temporary dev exposure |
| `samba_setup.sh` | Bootstrap Samba share steps on a Linux box |
| `tmux_mexc_dnx_logger.sh` | Attach tmux panes that tail MEXC/DNX log files |
| `kill_mexc_ticker_tunnel_processes.sh` | Clean stale SSH tunnel processes for ticker feeds |
| `test_server_port_8020.py` | Minimal FastAPI smoke test on port 8020 |

## Generic utilities (`util_*`)

Reusable snippets copied into larger projects so helpers do not multiply across repos:

| Script | Purpose |
|--------|---------|
| `util_cli_input.py` | Prompted CLI input patterns |
| `util_flask_client_info.py` | Log Flask request client metadata |
| `util_token_decimals.py` | ERC-20 decimal lookup helper |
| `util_terminal_colored_logs.py` | Colourised log lines in terminal tools |
| `util_terminal_title.py` | Set terminal window title from Python |
| `util_textual_buttons_demo.py` | Textual widgets sandbox (buttons, switches, radios) |
| `util_graceful_shutdown.py` | Signal-aware shutdown boilerplate |
| `util_site_indexing_check.py` | Crawl a site map and report HTTP status per URL |
| `util_mp4_to_gif.py` | ffmpeg wrapper for short screen captures |
| `util_network_card_speed.py` | Report NIC link speed from `ethtool` |

## Discord experiments

The `discord/` folder holds two Node scripts (`discord_reply_group_detector.js`, `discord_visual_grouping_test.js`) for discord.js reply-grouping experiments — prototypes only, not production bots.

## Quick start

```bash
git clone  # private repo — see REPOS.md
cd useful-scripts

python3 mexc_proto_auto_setup.py
python3 mexc_gate_dual_orderbook_dashboard.py
# or
python3 mexc_protodepth_tui_single.py
```

Always run from the repo root so `generated_proto/` and `mexc_proto_loader.py` resolve.

## Sibling products

| Repo | Relationship |
|------|----------------|
| [mexc_trading_app](https://github.com/logicencoder/mexc_trading_app) | Production MEXC trading dashboard and bots |
| [mexc_trading_app-overview](https://github.com/logicencoder/mexc_trading_app-overview) | Portfolio page for that app |
| [cex_dex_arb_app](https://github.com/logicencoder/cex_dex_arb_app) | Full CEX/DEX arbitrage product with its own embedded protobuf copy |
| [multi-coin-monitor-overview](https://github.com/logicencoder/multi-coin-monitor-overview) | Fleet screening dashboard built from similar feed code |

**useful-scripts** keeps the **canonical standalone copy** for one-off monitors and protobuf maintenance — prototypes here may graduate into those products later.

Private code: [useful-scripts](https://github.com/logicencoder/useful-scripts)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
