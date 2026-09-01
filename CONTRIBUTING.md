# Contributing to Foliate for Windows

> **Note:** This file has been modified using AI.

Thank you for your interest in contributing to Foliate for Windows! This project is a Windows port of the [Foliate](https://github.com/johnfactotum/foliate) e-book reader, built with Tauri 2 and WebView2.

## Code of Conduct

By participating in this project, you agree to abide by our [Code of Conduct](CODE_OF_CONDUCT.md) (if present) or the general principles of respectful, inclusive collaboration.

## How to Contribute

### Reporting Bugs

Before opening an issue, please check if it has already been reported.

**Bug reports should include:**
- Windows version (10/11, build number if possible)
- Foliate4w version (from About dialog or `CHANGELOG.md`)
- Steps to reproduce
- Expected vs actual behavior
- Screenshots or screen recordings if applicable
- Relevant log output (from `%LOCALAPPDATA%\Foliate\logs\` for installed, `Data\logs\` for portable)

### Suggesting Features

Feature requests are welcome! Please:
1. Check existing issues and the [README.md](README.md#current-features) for planned/unfinished features
2. Explain the use case and why it would benefit users
3. Note if it exists in upstream Foliate — we prioritize parity with the original

### Pull Requests

**We accept PRs for:**
- Bug fixes
- Feature parity with upstream Foliate (see [README#Relationship to Foliate](README.md#relationship-to-foliate))
- Windows-specific improvements (packaging, installer, WebView2 integration)
- Accessibility improvements
- Performance optimizations
- Documentation updates

**Before submitting a PR:**
1. Fork the repository and create a feature branch
2. Run `npm run check` (frontend build + `cargo check`) — CI must pass
3. Test on Windows (the only supported platform)
4. Update `CHANGELOG.md` under `## Unreleased` with your changes
5. Ensure no new warnings from `npm run lint` or `cargo check`

**PR Title Format:**
- `fix: <description>` — bug fixes
- `feat: <description>` — new features
- `docs: <description>` — documentation only
- `refactor: <description>` — code restructuring
- `chore: <description>` — build, deps, tooling

### Development Setup

**Prerequisites:**
- Windows 10/11 x64 (required for WebView2 and packaging)
- Node.js 24+
- Rust stable (MSVC toolchain)
- Microsoft WebView2 Runtime (usually pre-installed on Windows 11)

**Quick Start:**
```bash
git clone https://github.com/vihaanvp/Foliate4w.git
cd Foliate4w
npm ci
npm run web:dev          # Frontend only (http://localhost:1420)
# In another terminal:
npm run dev              # Full Tauri app (requires Windows)
```

**Commands:**
| Command | Purpose |
|---------|---------|
| `npm run web:dev` | Vite dev server (browser, no Tauri) |
| `npm run web:build` | Build frontend to `dist/` |
| `npm run check` | `web:build` + `cargo check` |
| `npm run dev` | Full Tauri app with hot reload |
| `npm run build` | Release build (portable + installers) |

**Note:** The maintainer machine has no local Rust toolchain. Rust changes are verified by GitHub Actions on `windows-latest`. Full packaging/runtime verification only runs meaningfully on Windows.

### Architecture Notes

**Do not modify:**
- `web/public/foliate-js/` — Vendored upstream engine (MIT). Treat as read-only. Engine fixes go upstream to `johnfactotum/foliate-js`, then re-vendored.
- `web/public/foliate-js/` tooling (ESLint, Rollup configs) — Upstream manages its own.

**Key constraints:**
- **One HTML page** (`web/index.html`) serves library + all reader windows. No router.
- **Range-byte IPC** — `NativeBookFile`/`NativeBookSlice` in `main.js` implement `Blob`/`File` via `invoke('read_book_range', {path, begin, end})`. Never pass whole files through IPC.
- **Stateless Rust shell** — No Rust-side database. Library lives in IndexedDB (`web/library.js`). Cross-window sync via `BroadcastChannel`.
- **Book identity is content-derived** — `getBookIdentity()` uses EPUB identifier (SHA-256) or fingerprint fallback. Moving/renaming files must not break library records.
- **Minimal Rust shell** — Only `serde`, `tauri`, `windows-sys`. No Tauri plugins. Binary size is a stated goal.
- **Portable vs installed** — `lib.rs::prepare_runtime` checks for `portable.flag`. Portable edition must fail loudly if its directory isn't writable (never silently fall back to `%APPDATA%`).

### Accessibility

Accessibility is **enforced, not optional**. Preserve:
- Skip link in `RootLayout.tsx`
- `prefers-reduced-motion` guards on all animations
- `aria-hidden` on decorative SVG
- `scroll-margin-top` on anchored sections
- Focus-visible gold outline
- English-first, Chinese-opt-in localization (see `chineseText` map in `main.js`)

### License

By contributing, you agree that your contributions will be licensed under the project's license (GPL-3.0-or-later, see [COPYING](COPYING)).

### Questions?

Open a GitHub Discussion or issue. For Windows-port-specific issues, use this repository's tracker. For upstream Foliate or `foliate-js` issues, report to their respective projects.