```
███████╗██████╗ ███████╗███████╗██████╗ ████████╗███████╗███████╗████████╗
██╔════╝██╔══██╗██╔════╝██╔════╝██╔══██╗╚══██╔══╝██╔════╝██╔════╝╚══██╔══╝
███████╗██████╔╝█████╗  █████╗  ██║  ██║   ██║   █████╗  ███████╗   ██║   
╚════██║██╔═══╝ ██╔══╝  ██╔══╝  ██║  ██║   ██║   ██╔══╝  ╚════██║   ██║   
███████║██║     ███████╗███████╗██████╔╝   ██║   ███████╗███████║   ██║   
╚══════╝╚═╝     ╚══════╝╚══════╝╚═════╝    ╚═╝   ╚══════╝╚══════╝   ╚═╝
```

[![arb](https://img.shields.io/badge/arb-package-05d9e8?style=flat-square)](https://github.com/MenkeTechnologies/arb)
![type](https://img.shields.io/badge/self--sourcing-dashboard-9b5de5?style=flat-square)
![license](https://img.shields.io/badge/license-MIT-ff2a6d?style=flat-square)

### `LIVE PING, DOWNLOAD, AND UPLOAD THROUGHPUT — MEASURED ON A TIMER, NOTHING PIPED IN`

> *"Watch your link's ping, down, and up redraw themselves every ten seconds."*

`speedtest` is an `arb` dashboard that monitors your internet connection: round-trip ping latency plus download and upload throughput, as reported by `speedtest-cli --simple`. It is **self-sourcing** — the spec runs its own measurement command on a timer, so you never pipe anything into it; open the dashboard and the numbers refresh on their own. It is an `arb` package — a dashboard for [`arb`](https://github.com/MenkeTechnologies/arb), the pipeline-to-TUI language on the fusevm bytecode VM.

### [`arb`](https://github.com/MenkeTechnologies/arb) &middot; [`arb-registry`](https://github.com/MenkeTechnologies/arb-registry)

---

## Table of Contents

- [\[0x00\] Overview](#0x00-overview)
- [\[0x01\] Install](#0x01-install)
- [\[0x02\] Usage](#0x02-usage)
- [\[0x03\] The Spec](#0x03-the-spec)
- [\[0x04\] How It Works](#0x04-how-it-works)
- [\[0xFF\] License](#0xff-license)

## [0x00] OVERVIEW

`speedtest` turns a periodic `speedtest-cli --simple` run into a live terminal dashboard. Each measurement cycle emits the three metrics that describe a connection — ping, download, upload — and the dashboard re-renders them in place. It exists so a link's health is a glance away in the same terminal you already live in, with no cron job, log file, or manual re-run.

## [0x01] INSTALL

```sh
arb install speedtest
arb search speedtest
```

Installs from the [`arb-registry`](https://github.com/MenkeTechnologies/arb-registry) git index.

## [0x02] USAGE

```sh
arb speedtest              # run the TUI dashboard (self-sourcing)
arb speedtest --serve      # serve the dashboard in a browser
arb speedtest --html > speedtest.html   # emit a static HTML dashboard
```

## [0x03] THE SPEC

```arb
# speedtest — Ping/download/upload throughput from speedtest-cli, self-sourced from `speedtest-cli --simple`.
! speedtest-cli --simple every 10s
list .speedtest
source .speedtest { in }
grid .speedtest -row 0 -col 0
```

- `! speedtest-cli --simple every 10s` is the **self-source**: `arb` runs `speedtest-cli --simple` every ten seconds and feeds each run's stdout into the `.speedtest` stream, so no external pipe is required.
- `list .speedtest` declares a **list widget** bound to that stream, rendering each output line (`Ping:`, `Download:`, `Upload:`) as a row.
- `source .speedtest { in }` is the **query pipeline** for the widget: `in` passes the incoming self-sourced lines straight through, unshaped, into the list.
- `grid .speedtest -row 0 -col 0` **places** the widget at row 0, column 0 of the dashboard grid.

## [0x04] HOW IT WORKS

The `!` self-source drives the whole dashboard: on its `every 10s` timer, `arb` executes `speedtest-cli --simple` and pushes the command's stdout into the `.speedtest` stream. The `source .speedtest { in }` pipeline shapes that stream — here just passing it through — and the `list` widget renders the result, redrawing at row 0 / column 0 on every cycle. `arb` lowers the pipeline compute to the fusevm bytecode VM with Cranelift JIT, so the shaping and rendering run as compiled code rather than an interpreted loop.

## [0xFF] LICENSE

MIT.
