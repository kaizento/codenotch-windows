# Contributing

This repository ships the **Windows** application in `windows/`. The macOS application in `Sources/`
is the upstream project — contributions to it go to
[vinzdg/codenotch](https://github.com/vinzdg/codenotch/blob/main/CONTRIBUTING.md), not here.

## Building

```powershell
cd windows
cargo build --release
.\target\release\codenotch.exe
.\target\release\codenotch.exe doctor    # what the app can see: credentials, data sources, hooks
```

Rust stable (MSVC) and the Visual Studio 2022 Build Tools with the C++ workload are the only
prerequisites; see the [README](README.md#building-from-source). `ui/notch.html` is embedded at build
time, so changes to the page need a rebuild, and a running `codenotch.exe` must be quit first.

## Before opening a PR

There are no automated tests in the Windows tree. Instead, check the change against the real thing:

- **Build is clean** — `cargo build --release` with no new warnings.
- **The pill still behaves** — hover opens the card, the card follows the pointer between cells and
  closes when the pointer leaves, drag moves the pill, × quits, tray items work.
- **Clicks pass through** — with the app running, a click in the empty area next to the pill must
  reach the application underneath. The quickest check is `WindowFromPoint` from PowerShell:

  ```powershell
  Add-Type @'
  using System;using System.Runtime.InteropServices;
  public struct PT{public int X,Y;public PT(int x,int y){X=x;Y=y;}}
  public class W{[DllImport("user32.dll")]public static extern IntPtr WindowFromPoint(PT p);
  [DllImport("user32.dll")]public static extern uint GetWindowThreadProcessId(IntPtr h,out uint pid);}
  '@
  $h=[W]::WindowFromPoint((New-Object PT(2300,700))); $p=0; [void][W]::GetWindowThreadProcessId($h,[ref]$p)
  (Get-Process -Id $p).ProcessName    # expected: the app under the notch, not msedgewebview2
  ```

  Use a point inside the window's rectangle but outside the pill (the rectangle is logged in
  `run.log` as `notch placed ... pos=(x,y) size=(w x h)`).
- **DPI** — if you touched geometry, try at least two scale factors (100 % and 125 % or 150 %).
  `run.log` records the DPI report and the zoom correction on every start.
- **Strings** — anything user-visible goes through `i18n.rs` (tray, window labels) or the
  `STRINGS` table in `notch.html` (card), with at least `en` and `ru` filled in. `en` is the fallback.
- **Comments explain why**, not what: a hidden constraint, a bug the code works around, a decision
  that would otherwise look arbitrary. If removing a comment would not confuse the next reader, it
  should not be there.
- **Update `CHANGELOG.md`** under *Unreleased*.

## Upstream etiquette

The fork exists to be mergeable. Keep changes to `windows/` small and self-contained, do not touch
`Sources/`, and prefer a constant or a config key to a fork of a function. When upstream changes
`windows/`, merge it by hand and re-check the list above.

A change that is useful to everyone — not tied to the Russian locale or to this fork's choices —
belongs upstream first: open a PR against
[vinzdg/codenotch](https://github.com/vinzdg/codenotch) and merge it here from there.

## Reporting a bug

Use the [issue template](.github/ISSUE_TEMPLATE/bug_report.md). Include the output of
`codenotch.exe doctor` and the tail of `%APPDATA%\codenotch\run.log`; neither contains tokens.
