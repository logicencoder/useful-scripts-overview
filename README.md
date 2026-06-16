# useful-scripts

Personal **Logic Encoder** script toolbox — MEXC and Gate.io market-data prototypes, protobuf setup helpers, balance and order testers, chain gas checks, fleet smoke scripts, and small `util_*` snippets collected while building trading and monitoring stacks. These are **local scripts** you run from a checkout when debugging depth feeds, validating API keys, triaging a stuck tunnel, or proving a deploy before opening six terminal tabs — not a shipped product, WordPress plugin, or fleet dashboard.

The private tree lives in [logicencoder/useful-scripts](https://github.com/logicencoder/useful-scripts). This public repo is a readable map of what exists and when to reach for each script — no API keys, hostnames, or operator paths.

After the 2026 cleanup every script uses a clear prefix (`mexc_*`, `gate_*`, `util_*`, `eth_*`, `wp_*`, `dnx_*`) and **one shared protobuf pipeline** so monitors stop breaking each other when MEXC updates WebSocket schemas on GitHub. Three **operator waves** (2026-06) added env-driven smoke, tail, and probe CLIs that wrap the same stacks production apps use — USM, CFMS, live-stats, arb, gas, shop, WordPress, and DNX tooling.

- **Single `generated_proto/` decoder tree** — all MEXC monitors import through `mexc_proto_loader.py`.
- **Refresh from upstream** — `mexc_proto_auto_setup.py` downloads schemas and recompiles after API changes.
- **Terminal and browser depth tools** — Textual TUIs, FastAPI dashboards, and plain loggers for the same binary feeds production apps use.
- **Order and balance sandboxes** — isolated scripts to test listen keys, single-order guards, and immediate buys without the full trading stack.
- **Fleet operator arsenal** — wave 1–3 smoke scripts grouped by incident type (deploy, trading desk, WP ops).

## Tech stack

| Layer | Technologies |
|-------|--------------|
| Languages | Python 3, Bash, small Node snippets in `discord/` |
| MEXC feeds | Spot V3 protobuf WebSocket, REST where scripts need bootstrap |
| Gate feeds | WebSocket depth and trades (no MEXC protobuf stack) |
| UI | Textual + Rich TUIs, FastAPI + uvicorn for local dashboards |
| Protobuf | `mexc_api_schema_from_github/` (`.proto` from [mexcdevelop/websocket-proto](https://github.com/mexcdevelop/websocket-proto)), `generated_proto/` (`*_pb2.py`) |
| Chain | web3.py, eth-defi UniV3 quoter, local Geth HTTP/WS, Dynex JSON-RPC |
| Smoke / probes | `requests`, `pyyaml`, `websockets`, `aiohttp`, `colorama` (see `requirements-smoke.txt`, `requirements-wave2.txt`) |
| Config | `config/mexc_keys.json` from `mexc_keys.example.json` — gitignored, local only |
| Hosting | Manual runs from a developer checkout on WSL or Linux; WP wave scripts may use SSH + wp-cli |

## Operator arsenal — three waves

All wave scripts run from the private repo root (`cd useful-scripts`). Configure targets through **environment variables** — nothing sensitive is committed. Install deps once: `pip install -r requirements-smoke.txt` (wave 1), then `pip install -r requirements-wave2.txt` when you need spread/quoter or shard probes.

### Wave 1 — fleet health and deploy smoke

Reach for these after a reboot, tunnel change, or USM reload when you need exit codes instead of clicking through dashboards.

| Script | Purpose |
|--------|---------|
| `usm_fleet_smoke.py` | `USM_SERVICES_YAML` TCP ports + optional `USM_URL` |
| `usm_multi_log_tail.sh` | Tail logs for one `services.yaml` group |
| `cloudflared_tunnel_cleanup.sh` | List/kill duplicate `cloudflared` processes |
| `cfms_tunnel_matrix_smoke.py` | `CFMS_URL` status + ingress HEAD probes |
| `mexc_live_stats_health.py` | `MEXC_TD_URL` monitoring + dead symbols |
| `gate_live_stats_health.py` | `GATE_TD_URL` monitoring + dead symbols |
| `mexc_reload_symbols_cli.py` | POST reload + poll until monitoring stabilizes |
| `geth_rpc_smoke.py` | `GETH_HTTP` / optional `GETH_WS` block + sync |
| `dynexd_rpc_smoke.py` | `DYNEX_NODE_URL` getinfo + block height |
| `wp_dynex_ingest_smoke.py` | `WP_URL` + `DYNEX_WP_API_KEY` ingest endpoints |
| `eth_gas_stack_smoke.py` | `GAS_PY_URL` + `GAS_NODE_URL` full gas stack |
| `le_shop_bridge_smoke.py` | WP applications + webhook + admin stats |
| `le_crypto_store_smoke.py` | `/shop/products` + `/admin/stats` |
| `util_telegrt_cleanup.py` | `TELEGRT_DIR` / `SOL_PUMP_HOME` Chrome cleanup |
| `util_sol_pump_sync_sol.sh` | `SOL_PUMP_SRC` + `SOL_PUMP_DST` rsync deploy |
| `wp_theme_deploy_purge.sh` | `WP_THEME_SRC` + `WP_THEME_REMOTE` + cache purge |

When a live-stats reload looks stuck, pair `mexc_reload_symbols_cli.py` with `mexc_live_stats_health.py`. When WordPress Dynex plugins stop updating, run `wp_dynex_ingest_smoke.py` before touching plugin code.

### Wave 2 — trading desk, arb, and spread research

These scripts sit beside the production trading dashboard and arb scanner — they surface feed age, debug logs, scanner diagnostics, and on-chain spread math in the terminal.

| Script | Purpose |
|--------|---------|
| `util_arb_diagnostics_cli.py` | `ARB_URL` — scanner-feeds, hub, rank, parity, api-timing |
| `mexc_feed_health_cli.py` | `MEXC_TRADING_URL` connection flags + bot perf percentiles |
| `mexc_debug_log_tail.py` | Poll or `--ws` tail debug logs (`order`, `warn`, `error` filters) |
| `mexc_gate_scanner_feed_audit.py` | Scanner feed audit + optional CEX REST ping |
| `gate_reload_symbols_cli.py` | `GATE_TD_URL` reload twin of MEXC |
| `mexc_multiconn_shard_probe.py` | `MULTICOIN_COINS_JSON` protobuf WS shard probe (50 pairs/socket) |
| `dnx_cex_dex_spread_tui.py` | DNX MEXC/Gate vs UniV3 refresh board (`GETH_HTTP`) |
| `eth_univ3_quoter_ladder.py` | On-chain buy/sell USD ladder for one pool |
| `eth_swap_monitor_smoke.py` | `ETH_SWAP_MONITOR_URL` mode + stats checklist |
| `eth_rpc_provider_probe.py` | HTTP/WS latency table for RPC providers |

Shared helper: `util_univ3_quoter.py` (imported by ladder and spread board). When arb scanner feeds look empty but the UI still spins, run `util_arb_diagnostics_cli.py scanner-feeds --strict` before restarting the scanner process.

### Wave 3 — WordPress ops, DNX flow, and deep probes

Hostinger and research workflows that used to mean SSH plus half-remembered wp-cli commands, or opening three apps to validate one symbol.

| Script | Purpose |
|--------|---------|
| `hostinger_log_tail.sh` | `HOSTINGER_SSH` + `WP_REMOTE_ROOT` tail debug logs |
| `wp_sitemap_generate_ping.sh` | Remote wp-cli sitemap generate + search ping |
| `wp_le_settings_ops_smoke.py` | Remote le-settings flags + security log table counts |
| `wp_visitor_stats_remote_verify.py` | `WP_URL` public HTML tracking snippet check |
| `wp_indexnow_queue_status.py` | MEXC/Gate IndexNow queue depth via wp-cli |
| `dnx_mexc_flow_report.py` | `DNX_MEXC_DB` SQLite IN/OUT report (no interactive menu) |
| `dnx_swap_smoke.py` | `DNX_SWAP_URL` status, nonce, node health |
| `util_karaoke_operator_preflight.sh` | Node, ffmpeg, GPU, HF token before render |
| `util_karaoke_job_smoke.sh` | `KARAOKE_API` process job + poll until done |
| `util_cs_agent_model_benchmark.sh` | `CS_AGENT_URL` `/api/models` + `/api/test-one` batch |
| `eth_mev_builders_probe.py` | MEV builder init + bundle RPC reachability |
| `eth_mode2_backfill_cli.py` | Mode 2 stats from swap monitor + proof log tail |
| `mexc_ws_dashboard_probe.py` | Capture N messages from `/ws-dashboard` |
| `mexc_symbol_stats_probe.py` | Per-symbol debug + memory stats on live-stats |
| `mexc_multicoin_reload_cli.py` | `MULTICOIN_URL` reload coins + poll stats |
| `util_stored_coins_validate.py` | `MULTICOIN_COINS_JSON` schema + duplicate check |

Wave 3 WP scripts expect `HOSTINGER_SSH` and `WP_REMOTE_ROOT` unless noted otherwise. For Mode 2 backfill **execution**, the swap monitor process still owns `--mode2-backfill-blocks`; `eth_mode2_backfill_cli.py` reports status and tails `ETH_MODE2_PROOF_LOG`.

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
| `util_smoke_http.py` | Shared HTTP/TCP helpers for wave smoke scripts |

## Discord experiments

The `discord/` folder holds two Node scripts (`discord_reply_group_detector.js`, `discord_visual_grouping_test.js`) for discord.js reply-grouping experiments — prototypes only, not production bots.

## Quick start

```bash
git clone  # private repo — see REPOS.md
cd useful-scripts

pip install -r requirements-smoke.txt
pip install -r requirements-wave2.txt   # spread / quoter / shard probes

python3 mexc_proto_auto_setup.py
python3 usm_fleet_smoke.py
python3 mexc_gate_dual_orderbook_dashboard.py
```

Always run from the repo root so `generated_proto/` and `mexc_proto_loader.py` resolve.

## Sibling products

| Repo | Relationship |
|------|----------------|
| [mexc_trading_app](https://github.com/logicencoder/mexc_trading_app) | Production MEXC trading dashboard and bots |
| [mexc_trading_app-overview](https://github.com/logicencoder/mexc_trading_app-overview) | Portfolio page for that app |
| [cex_dex_arb_app](https://github.com/logicencoder/cex_dex_arb_app) | Full CEX/DEX arbitrage product with its own embedded protobuf copy |
| [multi-coin-monitor-overview](https://github.com/logicencoder/multi-coin-monitor-overview) | Fleet screening dashboard built from similar feed code |
| [universal-service-manager-overview](https://github.com/logicencoder/universal-service-manager-overview) | USM fleet orchestration — wave 1 smoke targets its `services.yaml` |

**useful-scripts** keeps the **canonical standalone copy** for one-off monitors, protobuf maintenance, and terminal operator tooling — prototypes here may graduate into those products later.

Private code: [useful-scripts](https://github.com/logicencoder/useful-scripts)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
