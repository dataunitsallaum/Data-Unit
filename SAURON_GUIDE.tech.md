---
slug: data-unit
name: Data & AI Unit dashboard
url: https://nice-sea-0604a7a10.7.azurestaticapps.net
updated: 2026-09-24
audience: admin
---
# Data & AI Unit dashboard: technical guide

## What the app is made of
The whole app is one file, `index.html`, about 450 KB. It has no build step, no server code and no database. The file holds the page layout, the styles, the content data, two canvas network maps and an inlined copy of the Microsoft sign-in library (MSAL browser 2.38.1). The library is most of the file size. The repo also holds `staticwebapp.config.json`, the GitHub deploy workflow, `README.md` and `CLAUDE.md`. `CLAUDE.md` is the accurate project handoff. Read it before any edit.

## Data sources
The dashboard reads no live data. It does not query Power BI, NetSuite, SharePoint or Supabase. Every figure, list, date and link is written into `index.html` by hand. The unit keeps a solutions tracker workbook (`Data-AI-Unit-Solutions-Tracker.xlsx`) outside the repo. Nothing links it to the page.

Apart from Microsoft sign-in, the only outbound call is the usage telemetry beacon. See "Usage telemetry".

## Content data objects
All content edits happen in a few objects inside the main `<script>` block.
- `UPDATES`: the notification feed. One array drives three things: the Latest updates cards, the bell panel and the glowing dots on the Current Work map. Each entry has `date`, `type` ("new" or "mod"), `net`, `scope`, `title`, `note`, `color`, an optional `url` and `nodes`. The code never reads `net`. The cards and the bell show only the first nine entries.
- `ROAD`: the ten rows of the 2026 roadmap table. Each row is name, departments, description, priority (`h`, `m` or `l`) and timeline.
- `LINKS`: project name to URL. A project with a link gets a ring and a ↗ and opens in a new tab.
- `DESC`: project name (or pillar key) to tooltip text.
- `groups`: the five pillars, keys `A` to `E`. Each has `label`, `color`, `projects` (Current Work) and `future` (Future Work). An optional `subgroups` list can nest child dots under a sub-hub. It is unused today.

Pillar keys: `A` Data Analytics & Power BI (#E2231A), `B` Automations (#0E8C6B), `C` Agents (#E07B1A), `D` Apps (#6D5BD0), `E` Tech Hold (#2563EB).

To add a project, add its name to the right `groups` array, add a `DESC` entry and add a `LINKS` entry if it has a URL. To remove one, delete it from all three.

The headline figures, delivery cards, strategy phases, success-target table and AI Operating Model view are plain HTML. They are edited directly in the markup.

## UPDATES feed rules
These rules come from `CLAUDE.md` and the code comments. Follow them on every edit.
1. `date` is the real calendar date the entry is added. Never copy the date from the entry above.
2. The feed holds nine entries at most. Add new entries at the top and delete the oldest at the bottom.
3. Each `nodes` id must match a Current Work dot exactly, in the form `"<pillar letter>:<label>"`. A wrong id makes nothing glow.
4. Only the Current Work map glows. The code skips the Future Work map.
5. `color` should match the pillar colour.

When a new app is added to the map, the owner's standing rule is to add it to `UPDATES` too, unless told otherwise.

## How the key figures are calculated
Nothing is calculated at run time. The headline figures are fixed text.
- "450+ hrs/month saved" matches the sum of the "Est. saved" figures on the seven delivery cards: 100 + 200 + 20 + 12 + 15 + 100 + 10 = 457. Keep the headline and the cards consistent if either changes.
- Support hours on the cards add up to 79 hours per month (25 + 30 + 4 + 20). The page shows no total.
- "3 organizations", "8 departments", "30+ solutions" and "6+ AI initiatives" are hand-set. They are not counts of the map dots.
- The count-up animation on the headline figures reads each `data-count` attribute. It only animates. It computes nothing.
- The bell badge shows `UPDATES.length` minus a number stored in the browser's local storage under `sallaum_seen`. Opening the panel or "Mark all read" stores the current length.

## Sign-in
Sign-in runs in the browser with MSAL. It is locked to the Sallaum Microsoft tenant.
- The Entra app registration is "PowerBI Refresh", a shared single-page-app registration. The client ID and tenant ID sit in the auth script in `index.html`.
- The page uses `loginRedirect` with the `openid` and `profile` scopes. Tokens are cached in session storage.
- The redirect URI is the page's own origin. Every host must be registered on "PowerBI Refresh" as a single-page-application redirect URI, with no trailing slash. A missing one fails with error AADSTS50011.
- Registered hosts today: the Azure site and https://data-ai-unit.netlify.app.
- Sign-in is enforced on every real host. It is skipped on `localhost`, `127.0.0.1`, `0.0.0.0` and `file://`, so the file previews locally without sign-in.
- The inline MSAL copy loads first. If it is missing, the page falls back to the Microsoft CDN and then jsDelivr.
- The footer "Sign out" link calls `__signOut()`, which runs `logoutRedirect`.

## Access and security model
- Any account in the Sallaum tenant can open the page. There is no per-user list and no roles.
- The gate is soft. MSAL controls who sees the rendered page, but the static HTML can still be downloaded without sign-in. The file holds no secrets. It does hold the Power BI share links and app URLs. Each of those has its own sign-in.
- A true edge-level gate needs the Azure Static Web Apps Standard plan (about $9 per month) plus a single-tenant Entra registration. It is not set up.
- `staticwebapp.config.json` rewrites every path to `index.html` and sets the `X-Content-Type-Options: nosniff` and `X-Frame-Options: SAMEORIGIN` headers.
- The dashboard is not on the Sallaum Enterprises portal list. Sauron treats it as admin-only.

## Usage telemetry
An inline beacon at the end of `index.html` mirrors the shared usage module from the App Usage Analytics project. It reports under the app name `dataunit`.
- It sends `session_start`, a `heartbeat` every 60 seconds while the tab is visible, and `session_end` when the page closes.
- It flushes the queue every 15 seconds to the `usage-ingest` edge function on the shared App Usage Analytics Supabase project.
- It sends the user's Entra ID token in the `x-entra-token` header. The server reads the identity from that token.
- It never reports anonymously. Without a token it holds up to 200 events and drops them when the page closes.
- Every failure is swallowed, so telemetry cannot break the page.

See "Known gotchas" for why it likely sends nothing today.

## Deployment
- Repo: `dataunitsallaum/Data-Unit`, branch `main`. Working copy on Wael's device: `Downloads\Data Unit App\sallaum-dashboard-azure`.
- Host: Azure Static Web Apps, Free plan, at https://nice-sea-0604a7a10.7.azurestaticapps.net.
- A push to `main` runs the GitHub Actions workflow `.github/workflows/azure-static-web-apps-nice-sea-0604a7a10.yml`. It uploads the repo root with no build. The site is live in about 1-2 minutes. Pull requests to `main` also get a preview deploy from the same workflow.
- The workflow authenticates with the repo secret `AZURE_STATIC_WEB_APPS_API_TOKEN_NICE_SEA_0604A7A10`.
- Netlify (https://data-ai-unit.netlify.app) is a separate copy. It is updated by manual drag-and-drop of `index.html`. It does not auto-deploy.
- The owner's standing rule: publish only when asked. Commit, push, then hard-refresh with Ctrl+Shift+R.
- The git identity is set per repo on this device. There is no global identity.

## Known gotchas
- **Telemetry likely sends nothing.** The beacon looks for a global `pca` object to get a token. The sign-in script declares `pca` inside a private function, so the beacon never sees it. With no token the beacon holds its events and drops them. Expect no `dataunit` rows in App Usage Analytics until this is fixed.
- **The bell badge misses new items once the feed is full.** The badge compares counts, not item ids. When the feed sits at nine and a new item replaces the oldest, returning visitors see no number.
- **The deploy workflow file can go missing.** A past folder move dropped it from the working tree while git still tracked it. If `git status` shows it deleted, restore it with `git checkout -- .github`. Never commit the deletion, or auto-deploy stops.
- **A stale copy sits in Downloads.** `C:\Users\wael.frashy\Downloads\index.html` is an old copy. The repo file is the only source of truth. Never copy from the old one.
- **The SMC Module link uses another tenant.** Its Power BI link carries a different tenant id from every other link. Sallaum accounts may be refused there.
- **The AI Operating Model view is hidden, not removed.** Its nav link is commented out. The `#aimodel` hash still opens it. Uncomment the `AI Model` link in the nav to show it again.
- **Naming drift.** The "R&D New Ventures" delivery card still uses the old pillar name. Its products now sit in Tech Hold. Pillar `D` was renamed from R&D to Apps.
- **The logo image is absent.** The page looks for `sallaum-logo.png`, which is not in the repo. The text fallback "Sallaum Enterprises" shows instead. This is intended.
- **README note on `logisoft-roadmap.html` is stale.** The Belgium Customs Automation link to that file is commented out in `LINKS`, so nothing 404s today.
- **Some linked projects are commented out.** The Market Monitoring Platform and Investment Portfolio Monitor links are hidden in `LINKS` on purpose. The owner prefers commenting links out over deleting them.
- **The footer date is fixed.** "Generated June 2026" is static text.
- **PowerShell commit messages.** On this device, avoid double quotes inside `git commit -m` in PowerShell. Use a single-quoted here-string instead.
