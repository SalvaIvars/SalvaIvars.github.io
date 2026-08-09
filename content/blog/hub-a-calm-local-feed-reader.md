---
title: "hub: a calm, local-first feed reader in Rust"
date: 2026-08-09
tags: ["rust", "tauri", "rss", "local-first", "desktop"]
---

I lost track of the web I actually wanted to read. Not the firehose — the blogs, the newsletters, the few feeds I genuinely look forward to. They lived scattered across inboxes, a dead Pocket, and bookmark folders nobody opens. Feed readers exist, but almost all of them are SaaS: an account, a cloud, telemetry, and a reading view that still looks like a website.

So I built my own. It's called **hub**, a desktop feed reader where everything stays on your machine. This is the story of why I made it, how it's built, and where it's going.

## The idea: a reader, not a service

The founding constraint was: *no cloud, no account, no tracking*. That's not a marketing slogan, it's the architecture. All my feeds, articles and searches live in a single SQLite database in the app's data directory. There's no backend to talk to and no API key to paste.

The second constraint was *calm*. I wanted the reading experience of a good book — not a news site. So posts are stored bare: just title, summary and link. When you open an article, the full content is extracted on demand and rendered as clean typography. The list is keyboard-first; opening an article marks it read. The reader has light, dark and sepia themes, and settings for font, size and column width.

![](/images/hub-light.png)

That's the light theme: sources and saved searches on the left, the reading list in the middle, calm and quiet.

## The stack: Tauri 2, not Electron

The app is [Tauri 2](https://tauri.app): a Rust backend inside the OS webview, with a React + TypeScript + Vite frontend. The main reasons were binary size and memory — a Tauri release app is a few megabytes where an Electron one is a hundred — and the fact that the webview is a system component, so the UI renders like a native macOS app.

One decision I'm glad I made early: the main window is created from Rust via `WebviewWindowBuilder`, not from `tauri.conf.json`. That lets me intercept navigation at the native level — any external URL opens in the system browser instead of navigating the app, and `window.open` / `target="_blank"` is denied. No popups, ever.

## Hexagonal architecture in a Cargo workspace

The backend is split into seven crates following ports & adapters, which is more discipline than a project like this strictly needs — and it paid off, because it made the pipeline testable without any I/O.

- `crates/domain` — pure models (`Article`, `Source`, `SmartFeed`, `SearchMode`…), zero dependencies.
- `crates/feeds` — feed discovery and parsing. The discoverer guesses the feed URL from any homepage you paste; parsing uses `feed-rs`.
- `crates/extractor` — clean-content extraction via `rs-trafilatura`.
- `crates/storage` — SQLite with FTS5 full-text search and a vector table, schema migrations via `PRAGMA user_version`.
- `crates/pipeline` — the async orchestration, with all ports injected. Tests use in-memory mock repos, no network.
- `crates/embeddings` — local semantic embeddings via `fastembed` loading an ONNX model.
- `crates/app` — the Tauri binary: commands, wiring, background tasks.

Domain models are serialized in snake_case and mirrored by hand in `src/types.ts` on the frontend. It's a bit of duplication, but the contract is explicit: change a Rust struct, update the mirror, run the serde round-trip test.

## The part I'm most proud of: smart feeds

Search is full-text over everything you've saved, powered by [SQLite FTS5](https://sqlite.org/fts5.html) (instant, offline). On top of that sit **smart feeds** — saved searches that can be semantic. Each article gets an embedding vector, computed *locally* with an ONNX model loaded by fastembed. No API, no network, no key. Three search modes:

- **BM25** — classic keyword ranking.
- **Vector** — cosine similarity over embeddings.
- **Hybrid** — BM25 + vector combined with [Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf).

The pipeline refreshes feeds on a timer (re-reading the interval from settings every cycle so a config change applies on the next run), and a backfill task computes missing embeddings a few seconds after startup. There's a slider to tune the vector similarity threshold.

![](/images/hub-smartfeed.png)

A saved search powered by embeddings, alongside plain keyword smart feeds in the sidebar.

## Two gotchas that cost me real time

- **`window.confirm` silently returns false in macOS WKWebView.** A "delete source" button I'd written with a native confirm just... didn't delete anything. No error, no console. I replaced it with inline two-click confirmation in React. Worth knowing before you build on webview.
- **`tokio::spawn` panics from Tauri's `setup`.** The setup hook runs on the main thread with no Tokio runtime, so background tasks must use `tauri::async_runtime::spawn`. The panic message ("no reactor running") only appeared at runtime.

Also, every Tauri command has to be registered in *two* places — the `generate_handler!` macro in Rust and the wrapper in `src/api/commands.ts` — with camelCase keys on the TS side (`sourceId` → `source_id`). Easy to forget one.

## Shipping it

Releases are driven by a [GitHub Actions](https://github.com/SalvaIvars/hub) workflow that builds installers on tag push: a DMG for macOS, an MSI for Windows, a `.deb` and an AppImage for Linux. Two real-world lessons came out of this:

- **No Intel macOS build.** The `ort-sys` crate (ONNX Runtime bindings) publishes no prebuilt binaries for `x86_64-apple-darwin`, so the build fails on Intel. I ship arm64 only and documented why.
- **Artifact names are coupled to the landing page.** The download buttons link to `releases/latest/download/hub.dmg`, `hub-Setup.msi`, `hub-amd64.deb`. Rename a file in the workflow and you must update the site too.

The landing page is a static, no-build site in the repo's `docs/` folder, published with GitHub Pages. It's deliberately tiny — plain HTML/CSS/JS, [pi.dev](https://pi.dev)-inspired — with the screenshots, the "what hub doesn't do" list, and per-platform install commands.

## Download it

The app is free and open source (MIT):

- Landing page: <https://salvaivars.github.io/hub/>
- Releases (DMG, MSI, DEB, AppImage): <https://github.com/SalvaIvars/hub/releases>
- Source: <https://github.com/SalvaIvars/hub>

## What's next — and what I'd do differently

It's honestly good enough that I use it daily, which was the whole goal. What it doesn't have: no syncing between machines (deliberate — syncing would mean a server, and that breaks the "no cloud" constraint; maybe a future OPML-export-then-import workflow), no mobile client, and the first-run embedding backfill downloads the ONNX model once, which is slow on a cold start.

If I restarted it today I'd spend less time on generic settings plumbing and more on the reading experience, and I'd write the frontend state model before the UI instead of discovering the "everything is a view" rule through bugs. But the core bet — local-first, calm, and mine — feels right. The internet I actually want to read lives in one place now: a file on my disk.

![](/images/hub-dark.png)

Dark theme, an extracted article open in the reader.
