# Project notes for Claude Code

Single-file static web app (`index.html`) for Rutgers Business School–New Brunswick students to track graduation progress. Deployed on Vercel as a static site. UI language: Chinese, English or Spanish (course names and RBS terms like major / concentration stay in English). All UI text lives in the `I18N` dictionary (`zh` / `en` / `es`) and is read with `t(key, ...args)`; add every new string to all three languages. Default: saved choice (`localStorage` `rbs-nb-lang`) > browser language (`zh-*` → Chinese, `es-*` → Spanish, else English). Missing keys fall back to English.

## Key rules encoded (verify before changing)
- Requirements follow the FA26 advising slides first, then myRBS pages. Accounting & AIS has **no electives**; Foundational Core has no Computer Applications.
- Two Business Core versions: Fall 2025-and-earlier (39 cr) vs Fall 2026 (36 cr), chosen by the user.
- Only one course may double count between two RBS majors; concentration courses may not double count with a major (except Fixed Income & Credit Analysis inside Finance, Professional Selling inside Marketing, and 33:390:435 for Real Estate + Finance).
- Finance concentration ("390 CERT") can take 33:390:380 / 400 / 420; most other 33:390 electives are Finance-majors-only (2024–26 catalog). A B in 33:390:300 is required to take any other Finance course.
- Graduation minimum 120 credits; 19-credit-per-term warning.

## Storage
- `window.claude` exists only inside a Claude artifact; elsewhere the page falls back to `localStorage` key `rbs-nb-tracker-v1`. Keep that fallback working.
- Google Drive sync (`drive*` functions): GIS token client + Drive REST `appDataFolder/tracker.json`, scope `drive.appdata` only — never widen it. Last-writer-wins by `state.rev` on connect; afterwards localStorage first, upload after a 2 s pause. 401 → retry token with `prompt: ''`, else show "重新连接". Hidden while `GOOGLE_CLIENT_ID` is the placeholder and inside Claude artifacts. localStorage keys: `rbs-nb-drive-v1` (connected flag + fileId), `rbs-nb-drive-token` (short-lived access token).

## Transcript import
- `parseTranscript` / `planImport` / `applyImport` (pure) + `pdfTextFromPages`; pdf.js 3.11.174 loads from cdnjs only when a PDF is uploaded (SRI pinned, blob worker). Everything runs in the browser.
- Decisions from the user: PR flag `E` → no credit (other credits, 0 cr); `PA`/`P` → done but warn (C-or-better rule); no grade in a future term → planned for that term; current term → in progress; `W`/`INC` skipped; below C not done. Financial Management below B warns. Business Calculus / Calc I → CALC slot. Courses not in `CAT` → other credits.
- Never store name, student number or grades. Test with fabricated transcripts only; never commit a real transcript.

## SAS Core suggestions
- `CORE_DATA` in index.html (between the CORE-DATA markers) maps course numbers to SAS Core codes, built from the public Rutgers Schedule of Classes API (classes.rutgers.edu/soc/api, no login). Refresh each term with `node tools/build-core-codes.mjs` and commit. Never scrape Degree Navigator.
- Suggestions only: shown as "可满足 …" tags and pre-checked in the 归入 SAS Core picker; the user confirms. Degree Navigator is authoritative.
- Exception to "everything in index.html": `tools/` holds maintenance scripts that are not loaded by the page.

## Analytics
- Cloudflare Web Analytics only (cookieless page views, to count visitors). Token in `CF_ANALYTICS_TOKEN`; loads only on the production hostname and never inside Claude artifacts. Never send course progress or any user data to analytics, and keep `privacy.html` in sync if this changes.

## Conventions
- No build step, no frameworks. Keep everything in `index.html`. The one exception is `privacy.html` (Google OAuth needs a standalone privacy-policy URL, linked from the home page footer); keep it in sync with what the app actually stores and the scopes it requests.
- Show course names alongside course numbers everywhere.
- Test logic by extracting the `<script>` and running it with Node (`module.exports` at the bottom exposes the pure functions).

## SEO
- `robots.txt`, `sitemap.xml` and `og.png` (1200×630 share image, fake data only) sit next to `index.html`. Head has title, description, canonical, Open Graph and JSON-LD; `page_title` / `page_desc` i18n keys update the title and description per language. Update `sitemap.xml` `lastmod` when content changes meaningfully.
