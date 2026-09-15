<div align="center">

# Codenotch for Windows

**A small black pill on the right edge of the screen that shows how much of your Claude Code,
Codex and Antigravity limits you have used — and whether Claude is still working.**

[![Windows build](https://github.com/kaizento/codenotch-windows/actions/workflows/windows-build.yml/badge.svg)](https://github.com/kaizento/codenotch-windows/actions/workflows/windows-build.yml)
![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11%20x64-0078D4)
![Rust](https://img.shields.io/badge/rust-stable%20(MSVC)-orange)
![Tauri](https://img.shields.io/badge/tauri-2.11-24C8DB)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

<img src="windows/docs/screenshot-card.png" alt="The pill on the right edge with the hover card open: session and weekly windows, reset times, live sessions" width="420">

[Русская версия](README.ru.md)

</div>

This is a fork of [vinzdg/codenotch](https://github.com/vinzdg/codenotch). The upstream project is a
macOS application (`Sources/`); its Windows port (`windows/`, Rust + Tauri 2 / WebView2) is what this
repository builds, ships and maintains. The macOS code is kept in the tree only so upstream changes can
be merged; nothing here modifies it.

**What this fork adds:** a Russian locale, a close button on the card, a window hit region so the
transparent part of the window no longer swallows clicks, larger card type, the reset time as a chip,
and the Cursor cell switched off. Details in [What this fork changes](#what-this-fork-changes) and
[`CHANGELOG.md`](CHANGELOG.md).

## Contents

- [What it shows](#what-it-shows)
- [Reading the pill](#reading-the-pill)
- [Install](#install)
- [Using it](#using-it)
- [Claude Code hooks](#claude-code-hooks)
- [Configuration and data](#configuration-and-data)
- [Building from source](#building-from-source)
- [Repository layout](#repository-layout)
- [What this fork changes](#what-this-fork-changes)
- [Keeping up with upstream](#keeping-up-with-upstream)
- [Known limitations](#known-limitations)
- [Troubleshooting](#troubleshooting)
- [Contributing, security, conduct](#contributing-security-conduct)
- [License and credits](#license-and-credits)

## What it shows

One cell per provider, stacked top to bottom. A provider that is not installed or signed in simply gets
no cell.

| Cell | Where the number comes from | What you see |
|---|---|---|
| **Claude** | `GET https://api.anthropic.com/api/oauth/usage` with the OAuth token Claude Code keeps in `~/.claude/.credentials.json` | Current session and weekly windows with their reset times. On a 429 the poll backs off (60 s, doubling, capped at 15 min) and the deadline is persisted, so relaunching does not spend another attempt. A thin arc spins inside the ring while a Claude Code session is working and pulses amber when one is waiting on you. |
| **Codex** | `GET https://chatgpt.com/backend-api/wham/usage` with the session the Codex CLI keeps in `~/.codex/auth.json` (read only, never refreshed); falls back to the `rate_limits` snapshot in the newest rollout log | Live 5-hour and weekly windows on paid plans, a monthly window on free; otherwise the last snapshot, marked stale by its own timestamp. |
| **Antigravity** | The local `language_server` bridge (quota summary), then Google's Cloud Code API for licensed accounts, then a plain count of today's model turns | A percentage when one exists, a `~count` when it does not — never an invented number. |
| Cursor | — | **Off in this build.** `CURSOR_ENABLED = false` in `windows/codenotch/src/main.rs`; set it to `true` and rebuild to get the cell back. |

Every adapter reads what the owning tool itself reads from; those endpoints are internal and can
change without notice. A failure degrades to a visible state — stale, needs sign-in, waiting for the
first reading — rather than a made-up percentage.

## Reading the pill

<img src="windows/docs/screenshot-pill.png" alt="Collapsed pill: two rings with percentages" width="110" align="right">

- **Ring** — the most constrained window of that provider, i.e. the one that will stop you first.
  Green below 50 %, yellow from 50 %, red from 80 %.
- **Percentage** under the ring — the same window. `~` in front means the value is derived, not
  reported by the vendor.
- **Dimmed cell** — the reading is stale: older than five minutes, or the provider said so.
- **Thin spinning arc** inside the Claude ring — a Claude Code session is working right now.
  **Pulsing amber ring** — a session is blocked waiting for you (a permission prompt, a question).
- **Hover** any cell for the card: every limit window with its bar, the reset time as a chip
  (`Resets in 12 min`, `Resets at 14:00`, `Resets Sun 21:00`), and live Claude Code sessions by name.
- **Close button** (×, top right of the card) quits the application.

<br clear="right">

## Install

### Download

Take `codenotch.exe` and `codenotch-hook.exe` from the [latest release](../../releases/latest) and put
them in any folder you keep (for example `%LOCALAPPDATA%\Codenotch\`). Run `codenotch.exe` — the pill
appears on the right edge of the primary monitor, vertically centred, and a tray icon appears.

Requirements: Windows 10 or 11, 64-bit, and the
[WebView2 runtime](https://developer.microsoft.com/microsoft-edge/webview2/) — already part of
Windows 11 and of any machine with Edge or Office.

> **SmartScreen.** The executables are not code-signed. On first run Windows shows
> "Windows protected your PC" — click *More info → Run anyway*. If you would rather not trust an
> unsigned binary, [build it yourself](#building-from-source); the whole build is four minutes.

Only one instance runs at a time: a second launch is refused and the running one shows a notice on the card — useful to remember right after a rebuild.

### Build from source

See [Building from source](#building-from-source). The result is the same two executables.

## Using it

| Action | Effect |
|---|---|
| Hover the pill | Opens the card for the cell under the pointer; it follows the pointer between cells and closes 250 ms after the pointer leaves. |
| Click a cell | Opens that provider's usage page in the browser. |
| Press and drag the pill up or down | Moves it along the right edge; the position is remembered (`notch_y` in `config.json`). |
| × in the card | Quits. Start again from the executable or the tray-installed autostart. |
| Tray icon (left or right click) | The menu below. |

**Tray menu**

- **Install hooks / Uninstall hooks** — wires or unwires Claude Code, see [Claude Code hooks](#claude-code-hooks).
- **Language** — Auto, 中文, English, 日本語, 한국어, Русский. *Auto* follows the Windows display language.
- **Refresh now** — polls every provider immediately and reloads provider marks.
- **Reset position** — puts the pill back to the vertical centre of the primary monitor.
- **Open data folder** — `%APPDATA%\codenotch` in Explorer.
- **Start with Windows** — writes `HKCU\Software\Microsoft\Windows\CurrentVersion\Run\Codenotch`
  pointing at the current executable with `--silent`: the app waits in the background and shows the
  pill once a session starts. Untick to remove the value. The path is the executable's location at
  the time you ticked it, so move the file and tick again.
- **Quit**

## Claude Code hooks

The "is it working?" signal comes from Claude Code itself. **Install hooks** in the tray merges
`codenotch-hook.exe` into `~/.claude/settings.json` — a backup of the file is written first, and only
entries whose command contains `codenotch-hook` are ever touched, so your own hooks stay as they are.

The hook reports these events to the running app over `http://127.0.0.1:<port>/event`
(loopback only, port from `config.json`, default `48666`), in well under 5 ms:

| Claude Code event | Reported as |
|---|---|
| `SessionStart` | session started |
| `UserPromptSubmit`, `PreToolUse`, `PostToolUse` | working |
| `Notification` | waiting for you |
| `Stop` | done |
| `SessionEnd` | session ended |

A transcript watcher covers the same ground for sessions started before the app, and for the
desktop app. **Uninstall hooks** removes exactly what **Install hooks** added.

## Configuration and data

Everything lives in `%APPDATA%\codenotch\`:

| File | Purpose |
|---|---|
| `config.json` | Settings, see below. Created on first run. |
| `usage.json`, `codex.json`, `antigravity.json` | Last good reading per provider, so the pill is not blank after a restart. |
| `run.log`, `watch.log` | Application and transcript-watcher logs. No tokens are ever written to them. |
| `glyphs\` | Your own provider marks, if any. |

`config.json`:

| Key | Default | Meaning |
|---|---|---|
| `port` | `48666` | Loopback port the hook messenger posts to. |
| `lang` | `"auto"` | `auto`, `zh`, `en`, `ja`, `ko`, `ru`. Same as the tray's *Language*. |
| `notch_y` | `0.5` | Vertical position of the pill as a fraction of the monitor height. |
| `drag_enabled` | `true` | Whether the pill can be dragged. |
| `bar_x`, `bar_y`, `bar_w` | — | Legacy fields from the horizontal-bar layout; ignored by the pill. |

Edit the file while the app is closed; it is read at start.

**Provider marks.** The icons in the rings are the SVGs from
[`@lobehub/icons-static-svg`](https://github.com/lobehub/lobe-icons) (MIT), embedded unmodified —
see [`windows/codenotch/glyphs/NOTICE.md`](windows/codenotch/glyphs/NOTICE.md). Drop your own
`claude.svg`, `codex.svg`, `gemini.svg` (or `.png`) into `%APPDATA%\codenotch\glyphs\` to override.
The marks remain the trademarks of their owners.

**Self-diagnosis.** `codenotch.exe doctor` prints what the app can and cannot see: credentials,
data sources, icons, installed hooks.

## Building from source

Prerequisites:

- **Rust** (stable, MSVC target) — <https://rustup.rs>.
- **Visual Studio 2022 Build Tools** with the *Desktop development with C++* workload, for the linker.
- **WebView2 runtime** (see [Install](#install)).

```powershell
git clone https://github.com/kaizento/codenotch-windows.git
cd codenotch-windows\windows
cargo build --release           # ~4 min the first time, ~2.5 min after; both crates of the workspace
.\target\release\codenotch.exe
```

The result is `target\release\codenotch.exe` (~8 MB, the app) and `target\release\codenotch-hook.exe`
(the hook messenger). Two things to know:

- `ui/notch.html` is embedded into the binary at build time — any change to the pill or card needs
  a rebuild.
- A running `codenotch.exe` locks its file. Quit it (× on the card, or tray → Quit) before rebuilding,
  or the link step fails with *Access is denied*.

To keep the Rust toolchain off the system drive, set `RUSTUP_HOME`, `CARGO_HOME` and
`CARGO_TARGET_DIR` in the shell before `rustup-init` and `cargo build`; nothing in the project
assumes their location.

The [Windows build](.github/workflows/windows-build.yml) workflow builds the same thing on every
change under `windows/` and keeps the two executables as a downloadable artifact for two weeks.

## Repository layout

```
.
├── windows/                     The Windows application — everything this repository ships
│   ├── codenotch/               Tauri 2 app crate
│   │   ├── src/main.rs          Window, commands, hit region, drag, DPI correction
│   │   ├── src/tray.rs          Tray menu
│   │   ├── src/i18n.rs          Tray/window strings; the card has its own dictionary in notch.html
│   │   ├── src/usage.rs         Claude provider        src/codex.rs        Codex provider
│   │   ├── src/antigravity.rs   Antigravity provider   src/cursor.rs       Cursor provider (off)
│   │   ├── src/watcher.rs       Transcript watcher     src/state.rs        Session state engine
│   │   ├── src/server.rs        Loopback listener for the hook messenger
│   │   ├── src/hooks_install.rs settings.json merge    src/autostart.rs    Run-key autostart
│   │   ├── src/doctor.rs        `doctor` subcommand    src/glyphs.rs       Provider marks
│   │   ├── ui/notch.html        The pill and the card — one file, no framework
│   │   ├── glyphs/              Bundled provider marks (+ NOTICE.md)
│   │   └── tauri.conf.json      Window definition (340×460, transparent, always on top, no focus)
│   ├── codenotch-hook/          The <5 ms hook messenger Claude Code calls
│   ├── docs/                    Screenshots used by this README
│   └── README.md                The port's technical description (from its original author)
├── Sources/, Tests/, Makefile, project.yml, Scripts/, site/, docs/, TASKS.md
│                                The upstream macOS application, untouched; merged from upstream as is
├── .github/workflows/windows-build.yml   CI for the Windows app
├── .github/workflows/ci.yml, package.yml Upstream macOS workflows — disabled in this repository
├── CHANGELOG.md                 History of the Windows build
├── CONTRIBUTING.md · SECURITY.md · CODE_OF_CONDUCT.md
└── LICENSE                      MIT
```

## What this fork changes

Relative to the `windows/` tree in upstream `main`:

1. **Russian locale** — tray, window labels, provider notes (`i18n.rs`) and the card
   (`STRINGS.ru` in `notch.html`). `lang: "ru"` or tray → Language → Русский.
2. **Close button on the card.** The pill has no window chrome, so the only way out used to be the
   tray. The × calls a `quit_app` command that does what tray → Quit does. It reacts on `mousedown`
   rather than `click`: the card rebuilds its DOM on every usage event, the element under the pointer
   can vanish before `mouseup`, and a `click` then never fires.
3. **Window hit region.** The window is a 340 × 460 CSS px sheet, almost all of it transparent, pinned
   to the right edge — and a transparent pixel still belongs to the window. Every click in that area
   went to the pill instead of the application underneath, so buttons near the right edge of the
   screen stopped responding. `WS_EX_TRANSPARENT` would not help: the mouse is taken by the child
   WebView2 window, not by the Tauri window. `SetWindowRgn` on the parent clips the child with it. The
   region is the pill's rectangle while the card is collapsed and the bounding box of pill + card
   while it is open; the page reports both (`set_pill_rect`, `set_expanded`). Until the page has
   reported anything, the region is removed so the pill can never clip itself out of reach.
4. **Larger card type** and the **reset time as a chip** — readable on a 27" display at 125 %.
5. **Cursor cell off** by a single constant, see [What it shows](#what-it-shows).

Everything else — providers, session engine, DPI handling, tray — is the port as it is upstream.

## Keeping up with upstream

```powershell
git remote add upstream https://github.com/vinzdg/codenotch.git   # once
git fetch upstream
git merge upstream/main
```

Files this fork owns and that conflict on every upstream change: `README.md`, `CONTRIBUTING.md`
(take ours), and whatever upstream touched under `windows/` (merge by hand — the fork's changes are
small and listed above). `CHANGELOG.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `README.ru.md` and
`.github/workflows/windows-build.yml` do not exist upstream and merge cleanly.

The upstream macOS workflows (`ci.yml`, `package.yml`) are disabled in this repository's Actions
settings; they need a macOS runner and signing secrets that only the upstream maintainer has.

## Known limitations

- The pill lives on the **right edge of the primary monitor** only. The macOS app's four-edge
  placement is not ported.
- **Not code-signed**, so SmartScreen warns on first run and there is no auto-update. Check the
  [releases](../../releases) yourself.
- **Cursor** is switched off in this build (a constant, see above).
- **Hooks are not installed automatically** — tray → Install hooks, once.
- One pill per machine account: two Claude Code logins (`CLAUDE_CONFIG_DIR`) are not shown as two
  rings, unlike in the macOS app.

## Troubleshooting

| Symptom | What to check |
|---|---|
| No pill after start | The app may be in `--silent` mode with no session running — start a Claude Code session, or run the executable without arguments. Tray → Reset position if it was dragged off-screen on a display that has since changed. |
| Cell shows `—` | The provider needs a sign-in: hover for the note. Claude reads `~/.claude/.credentials.json`, Codex `~/.codex/auth.json`. Run `codenotch.exe doctor`. |
| Cell dimmed | Reading older than five minutes. Tray → Refresh now; on a 429 the back-off deadline is shown in `run.log`. |
| Clicks near the right edge of the screen go nowhere | Fixed in this fork (window hit region). If it comes back, `run.log` shows the rectangles the page reported. |
| Card opens at the wrong size on a scaled display | `run.log` records the DPI report and the zoom correction applied; the page also self-corrects with CSS zoom. Please open an issue with those lines and your scale factor. |
| Build fails with *Access is denied* on `codenotch.exe` | The app is running — quit it first. |

The log is `%APPDATA%\codenotch\run.log` (tray → Open data folder). It never contains tokens.

## Contributing, security, conduct

- [CONTRIBUTING.md](CONTRIBUTING.md) — building, what to check before a PR, upstream etiquette.
- [SECURITY.md](SECURITY.md) — what the app reads, how to report a vulnerability privately.
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
- Bugs in the Windows app: [open an issue](../../issues/new/choose). Bugs in the macOS app belong to
  [upstream](https://github.com/vinzdg/codenotch/issues).

## License and credits

[MIT](LICENSE).

- **Codenotch** — design, name and the macOS application: [Vinz](https://github.com/vinzdg),
  [vinzdg/codenotch](https://github.com/vinzdg/codenotch).
- **Windows port** (`windows/`) — [Im-Midi](https://github.com/Im-Midi/codenotch-windows), offered to
  upstream as its `windows/` tree; the session-detection engine originated in
  [Im-Midi/Pac-Man](https://github.com/Im-Midi/Pac-Man) (MIT).
- **Provider marks** — [`@lobehub/icons-static-svg`](https://github.com/lobehub/lobe-icons) (MIT);
  the marks are trademarks of their respective owners.
- **This fork** — [kaizento](https://github.com/kaizento).
