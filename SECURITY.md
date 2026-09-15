# Security policy

## What this application touches

Codenotch reads credentials that other tools already keep on your machine, and nothing else:

| Provider | What is read | Where |
|---|---|---|
| Claude | OAuth token Claude Code stores | `~/.claude/.credentials.json` |
| Codex | Session token the Codex CLI stores | `~/.codex/auth.json` (read only, never refreshed) |
| Antigravity | Local language-server bridge, then Google's Cloud Code API | local process / `~/.antigravity` |

Tokens are used solely to call each provider's own usage endpoint and are never written anywhere.
Persisted state (`%APPDATA%\codenotch\`) holds usage snapshots, the window position, the language
and a plain-text log; it never contains a token.

Claude Code hooks, when you install them from the tray, run `codenotch-hook.exe`, which posts the
event name to `http://127.0.0.1:<port>/event` on the local machine. The listener binds to
loopback only.

## Supported versions

Only the latest release on the `main` branch receives fixes.

## Reporting a vulnerability

Please **do not** open a public issue for security problems.

Use GitHub's private reporting: **Security → Report a vulnerability** on this repository
(<https://github.com/kaizento/codenotch-windows/security/advisories/new>). You will get an
acknowledgement within a week; fixes are published as a normal release with a note in
`CHANGELOG.md`.

Problems in the macOS application (`Sources/`) belong to the upstream project:
<https://github.com/vinzdg/codenotch/security>.
