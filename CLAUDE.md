# ConnecTHOR — project guide for Claude

Read this first. It replaces the chat history that built the project, so keep it current. When you finish a piece of work, update **Status** and **Open items** below.
For the background (what the owner asked for, the timeline, and why things were decided), read `HISTORY.md`.

## What it is
A single-page "situation monitor" inspired by PizzINT (pizzint.watch). A 3D globe shows geopolitical prediction markets, with a Pentagon Pizza Index / DOUGHCON panel, a "Nothing Ever Happens" (NEH) index, live trades, military aircraft and a news feed.

- Live site: https://dekabet.github.io/connecthor/ (GitHub Pages, deploys from `main`)
- Everything is in **one static file**, `index.html`: React 18 UMD, react-globe.gl 2.27.2 and in-browser Babel 7.25.6, all pinned from unpkg. There is no build step, no package.json and no backend (yet).
- Icons: `favicon.svg` is the source. `favicon.ico`, `favicon-32.png`, `apple-touch-icon.png` and `icon-512.png` are rendered from it, because Safari ignores SVG favicons.

## Layout of `index.html` (search for these markers)
| Section | Marker | Notes |
|---|---|---|
| Boot screen + failure message | `window.__booted` | Shows why the page failed instead of leaving it blank |
| News (RSS via rss2json, NewsAPI on localhost only) | `NEWS API CONFIG` | `NEWS_API_KEY` is public: **remove and rotate it** |
| Pizza index (modeled) | `PIZZA INDEX` | `SHOPS`, `DOUGHCON`, `modelSnapshot`, and `PIZZA_DATA_URL` for a real feed |
| Sample markets (offline fallback) | `const MARKETS=[` | |
| Polymarket: Gamma, Data API trades, CLOB history, WebSocket | `POLYMARKET` | `getJSON` = direct, then CORS proxies; `useLivePrices` = WebSocket |
| Kalshi / Manifold / PredictIt | `OTHER PREDICTION MARKETS` | `SOURCES`, `similarMarkets` (cross-venue matching) |
| adsb.lol military aircraft, GDELT coverage, Doomsday Clock | `OSINT LAYERS` | `DOOMSDAY` is a hand-maintained constant (85 s, Jan 2026) |
| NEH index | `NOTHING EVER HAPPENS INDEX` | Real-money venues only (`isUSD`) |
| Hotspots, Ticker, FlatMap (2D fallback), Boundary | `COMPONENTS` | |
| App state, effects, render | `function App()` | |

## Rules that matter (learned the hard way)
- **WebGL may be missing** (Brave without graphics acceleration). `HAS_WEBGL` switches to the 2D `FlatMap`. Never render `<Globe>` unguarded: an uncaught WebGL error blanks the whole page.
- **Keep globe data objects stable.** Arcs and rings are memoised on `shownKey` (the set of shown market IDs), and points are reused via `pointCache`. Recreating them on every price tick restarts their animations and makes markers re-grow.
- **The ticker swaps content only between loops** (`onAnimationIteration`) at `TICKER_PX_PER_S`. Never bind its duration or text straight to live data, or it jumps mid-scroll.
- **App state named `history`** (price history) shadows `window.history`. Use `window.history` explicitly.
- Polymarket's `outcomePrices`/`clobTokenIds`/`outcomes` are JSON-encoded strings; use `parseArr`. Kalshi prices are integer cents or `*_dollars` strings; use `kPrice`.
- Manifold is play money (Ṁ). Keep it out of the NEH index, the $ totals and volume-based rings/arcs.
- Respect `REDUCED_MOTION`, and skip polling when `document.hidden`.
- The page must degrade gracefully: every feed failure falls back to sample data or a labelled simulated view, never to a crash.

## How to test (no network to these APIs from the cloud sandbox)
The sandbox blocks unpkg and every data API. Install the libraries from npm into a scratch folder and serve them through Playwright `page.route`; mock each API with fixtures, and use `page.routeWebSocket` for the CLOB stream. Run Chromium from `/opt/pw-browsers/chromium`. Scenarios to cover: full mocks, offline (abort everything), and `--disable-webgl`. Also compile-check with `@babel/standalone` (classic React preset) before pushing.

**Nothing has been verified against the real Kalshi / Manifold / PredictIt / adsb.lol / GDELT servers from the sandbox.** Polymarket is confirmed live by the owner.

## Status (2026-10-17)
- PR dekabet/connecthor#1 and dekabet/connecthor#2 are merged and live.
- The owner wants to launch and share on X/Twitter for feedback.

## Open items (priority order)
1. **Pre-launch:**
   - Replace real pizza shop names with generic ones (the data is simulated).
   - Delete `NEWS_API_KEY` from the page (the owner should rotate it).
   - Add on-page PizzINT credit and a "for fun, not financial advice" footer.
2. **Owner check in a real browser:** which venues appear in the globe badge, whether Price stream shows LIVE, whether Flow shows real trades, the aircraft count, and the GDELT chart.
3. **Move to Cloudflare Pages + Functions** (planned):
   - An `/api/*` allow-listed proxy with edge caching per feed.
   - Secrets (NewsAPI) stored server-side.
   - The page tries `/api` first, then falls back to the current direct path.
   - `wrangler.toml`; the owner connects the repo in the Cloudflare dashboard (framework None, output `/`).
4. Real pizza data (a scheduled job feeding `PIZZA_DATA_URL`) if a lawful data source is found.

## Working conventions
- Single-file style. Match the existing compact code; comments explain *why*, not what.
- Commit messages: a summary line, then bullets. Open a PR into `main`; the owner merges.
- Never commit secrets.
