# Project notes for Claude Code

Single-file static web app (`index.html`) for Rutgers Business School–New Brunswick students to track graduation progress. Deployed on Vercel as a static site. UI language: Chinese, English or Spanish (course names and RBS terms like major / concentration stay in English). All UI text lives in the `I18N` dictionary (`zh` / `en` / `es`) and is read with `t(key, ...args)`; add every new string to both languages. Default: saved choice (`localStorage` `rbs-nb-lang`) > browser language (`zh-*` → Chinese, `es-*` → Spanish, else English). Missing keys fall back to English.

## Key rules encoded (verify before changing)
- Requirements follow the FA26 advising slides first, then myRBS pages. Accounting & AIS has **no electives**; Foundational Core has no Computer Applications.
- Two Business Core versions: Fall 2025-and-earlier (39 cr) vs Fall 2026 (36 cr), chosen by the user.
- Only one course may double count between two RBS majors; concentration courses may not double count with a major (except Fixed Income & Credit Analysis inside Finance, Professional Selling inside Marketing, and 33:390:435 for Real Estate + Finance).
- Finance concentration ("390 CERT") can take 33:390:380 / 400 / 420; most other 33:390 electives are Finance-majors-only (2024–26 catalog). A B in 33:390:300 is required to take any other Finance course.
- Graduation minimum 120 credits; 19-credit-per-term warning.

## Storage
- `window.claude` exists only inside a Claude artifact; elsewhere the page falls back to `localStorage` key `rbs-nb-tracker-v1`. Keep that fallback working.
- Google Drive sync (`drive*` functions): GIS token client + Drive REST `appDataFolder/tracker.json`, scope `drive.appdata` only — never widen it. Last-writer-wins by `state.rev` on connect; afterwards localStorage first, upload after a 2 s pause. 401 → retry token with `prompt: ''`, else show "重新连接". Hidden while `GOOGLE_CLIENT_ID` is the placeholder and inside Claude artifacts. localStorage keys: `rbs-nb-drive-v1` (connected flag + fileId), `rbs-nb-drive-token` (short-lived access token).

## Conventions
- No build step, no frameworks. Keep everything in `index.html`.
- Show course names alongside course numbers everywhere.
- Test logic by extracting the `<script>` and running it with Node (`module.exports` at the bottom exposes the pure functions).
