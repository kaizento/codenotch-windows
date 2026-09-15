<!-- Changes to the macOS app (Sources/) belong upstream: https://github.com/vinzdg/codenotch -->

**What changes and why**

**How it was checked** (see CONTRIBUTING.md — there are no automated tests in the Windows tree)

- [ ] `cargo build --release` in `windows/` is clean, no new warnings
- [ ] Hover / card / drag / × / tray still work
- [ ] Clicks in the empty area next to the pill reach the application underneath (`WindowFromPoint` check)
- [ ] Tried at a second display scale, if geometry was touched
- [ ] User-visible strings have `en` and `ru`
- [ ] `CHANGELOG.md` updated under *Unreleased*
