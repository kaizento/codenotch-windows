# Changelog

All notable changes to the **Windows build** (`windows/`) are recorded here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/);
versions follow [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

The macOS application in `Sources/` is the upstream project and keeps its own history at
[vinzdg/codenotch](https://github.com/vinzdg/codenotch).

## [Unreleased]

## [0.3.0-ru.1] — 2026-09-15

First release of this fork. Based on the Windows port shipped in upstream `windows/`
(Rust + Tauri 2, originally by [Im-Midi](https://github.com/Im-Midi/codenotch-windows)).

### Added
- **Russian locale** (`ru`) for the tray menu, window labels, provider notes and the hover card.
  Selectable in the tray under *Language*; `lang: "ru"` in `config.json`.
- **Close button** in the hover card (×, top-right). Exits the application the same way the tray's
  *Quit* does. It reacts on `mousedown`, not `click`: the card re-renders its DOM on every usage
  update, so the element under the pointer can vanish before `mouseup` and a `click` would never fire.
- **Window hit region.** The window is a 340 × 460 CSS px sheet of mostly transparent space pinned to
  the right edge, and a transparent pixel still belongs to the window — every click inside that
  rectangle was swallowed instead of reaching the application underneath. The window is now clipped
  with `SetWindowRgn` to the pill's rectangle while the card is collapsed, and to the bounding box of
  pill + card while it is open. The rectangles are the ones the page already reports; a new
  `set_pill_rect` command keeps the pill's rectangle current. While the page has reported nothing yet
  (startup, a reloaded WebView) the region is removed entirely so the pill can never clip itself out
  of reach.

### Changed
- Larger type in the hover card (window labels, "used" figures, notes) — the original sizes were hard
  to read on a 27" / 125 % display.
- Reset time rendered as a chip (dark pad, amber digits, tabular numerals) instead of plain text.
- Cursor cell disabled (`CURSOR_ENABLED = false` in `main.rs`). Flip the constant to bring it back;
  nothing else needs to change.
- `Cargo.toml` `repository` points at this repository.

[Unreleased]: https://github.com/kaizento/codenotch-windows/compare/v0.3.0-ru.1...HEAD
[0.3.0-ru.1]: https://github.com/kaizento/codenotch-windows/releases/tag/v0.3.0-ru.1
