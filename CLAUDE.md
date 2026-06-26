# Big5 Winter Series 2026 — Project Handoff

## What this is
A gym challenge web app where members log daily activity, earn points, and enter a prize draw. Runs for 8 weeks from 15 June 2026.

## Architecture
- **Frontend**: `index.html` hosted on GitHub Pages — `https://melitashelley.github.io/winterchallenge/`
- **Backend**: Google Apps Script (`Code.gs`) — `https://script.google.com/macros/s/AKfycbz5MiBgM6A6qlY9nJUvGszmtxulfdSmHHZ_WlxceYyWqh5kBOq-ifzOsxRWAOqce-6L/exec`
- **Database**: Google Sheet ID `1Z5cATP1hP-zwS1Hpi4obpc0JxTwMtwTZQYjfJ3o1GEg`
- **GitHub repo**: `https://github.com/melitashelley/winterchallenge`

## File structure
```
/
├── index.html          # Full PWA frontend (single file)
├── manifest.json       # PWA manifest for home screen install
├── Code.gs             # Google Apps Script backend (paste into Apps Script editor)
├── SETUP.md            # Step-by-step setup guide
├── CLAUDE.md           # This file
└── icons/
    ├── apple-touch-icon.png
    ├── favicon-32.png
    ├── favicon-48.png
    ├── icon-192.png
    └── icon-512.png
```

## How the API works
The frontend calls the backend via `fetch()` POST requests. No `google.script.run` — that only works when HTML is served from Apps Script itself.

```javascript
// In index.html — near top of <script>
var SCRIPT_URL = 'https://script.google.com/macros/s/.../exec';

function api(action, extra) {
  var payload = Object.assign({ action: action }, extra || {});
  return fetch(SCRIPT_URL, {
    method: 'POST',
    body: JSON.stringify(payload)
    // No Content-Type header = text/plain, avoids CORS preflight
  }).then(function(r) { return r.json(); });
}
```

Available actions: `getConfig`, `getLeaderboard`, `getMySubmissions`, `submitEntry`, `submitSpin`, `checkTodayStatus`, `getWeekStatus`

## Google Apps Script setup
- Code lives in `Code.gs` — paste into the Apps Script editor at `script.google.com`
- `doGet` returns a plain text "API is running" message (frontend is no longer served from GAS)
- `doPost` is the API router — reads `e.postData.contents`, parses JSON, dispatches by `action`
- To redeploy after changes: Deploy → Manage deployments → pencil icon → New version → Deploy
- The URL never changes when redeploying as a new version

## Google Sheet tabs
| Tab | Purpose |
|-----|---------|
| Submissions | Every activity log entry |
| Spins | Every wheel spin (wheel currently commented out) |
| Members | Member name list |
| Challenges | Weekly fitness challenges (name, points, color) |
| Wheel | Wheel segments — currently unused |
| Config | App settings (row 2: Challenge Start; row 3: Wheel Visible) |

## Key decisions made
- **Wheel feature is commented out** in index.html — can be re-enabled or redesigned later
- **Dates**: GAS stores dates as Date objects in sheets. Always use `Utilities.formatDate(date, tz, 'yyyy-MM-dd')` when comparing, never string comparison
- **CORS**: POST body sent as plain text (no Content-Type header) to avoid preflight OPTIONS request, which GAS doesn't handle
- **Mobile layout**: Uses `position: fixed; inset: 0; width: 100vw` + `height: 100dvh`. Desktop centred column only applies with `(min-width: 900px) and (hover: hover) and (pointer: fine)` to avoid triggering on phones
- **PWA**: `manifest.json` enables "Add to Home Screen" on mobile. Icons in `icons/` folder

## Deploying changes
**Frontend changes** (index.html, manifest.json, icons):
```bash
cd "/Users/melitashelley/Documents/Claude/Projects/Winter Challenge App"
git add -A
git commit -m "description of change"
git push origin main
```
GitHub Pages rebuilds automatically (~1 min).

**Backend changes** (Code.gs):
1. Paste updated `Code.gs` into Apps Script editor
2. Deploy → Manage deployments → pencil → New version → Deploy

## What's working
- Member name picker + localStorage persistence
- Activity logging (class attendance, steps, weekly challenges)
- Points calculation and leaderboard
- "Mine" tab showing personal history and week tracker
- Admin leaderboard sheet (`buildLeaderboardSheet()` function in Code.gs)
- PWA manifest + icons for home screen install

## What still needs testing
- CORS: verify `fetch()` from GitHub Pages to Apps Script URL works without errors (open browser dev tools → Network tab when submitting)
- Favicon and apple-touch-icon rendering on iOS
- Full end-to-end submit flow from the GitHub Pages URL
