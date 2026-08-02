# Sallaum Enterprises — Data & AI Unit Dashboard

> **Read this first.** This file is the full project context. It is auto-loaded by Claude Code,
> so a fresh session on any machine can pick up work immediately with no further explanation.

**Owner:** Wael Frashy — Lead, Data & AI Unit
**What it is:** A single self-contained `index.html` — an interactive dashboard mapping every
project the Data & AI Unit has delivered and plans to deliver, for Sallaum Enterprises management.

---

## 1. Where everything lives

| Thing | Location |
|---|---|
| **Source of truth** | `index.html` **in this repo** (single file, ~450 KB) |
| **GitHub repo** | https://github.com/dataunitsallaum/Data-Unit (branch `main`) |
| **Live site (Azure)** | https://nice-sea-0604a7a10.7.azurestaticapps.net |
| **Live site (Netlify)** | https://data-ai-unit.netlify.app (manual upload) |
| **Solutions tracker (Excel)** | `Data-AI-Unit-Solutions-Tracker.xlsx` — kept locally, not in this repo |

> ⚠️ **History note:** on the original Windows machine the canonical file was
> `C:\Users\wael.frashy\Downloads\index.html`, and publishing meant *copying* it into the repo
> folder. **On any new machine, drop that habit** — edit `index.html` in this repo directly.
> One file, one source of truth, no drift.

---

## 2. How to publish (this is the whole deploy process)

Azure Static Web Apps auto-deploys on every push to `main` (workflow in `.github/workflows/`).
Goes live in ~1–2 minutes.

```bash
git add index.html
git commit -m "Describe the change"
git push origin main
```

Then **hard-refresh** the live site (`Ctrl+Shift+R`) — browsers cache aggressively.

For **Netlify**, upload `index.html` manually (drag-and-drop). It is a separate origin and does
not auto-deploy.

**Windows/PowerShell gotchas** (hit repeatedly on the original machine):
- The Bash tool's PATH was sometimes broken (`cp`/`cat` missing, `git push` silently exiting 128).
  Running the same commands through **PowerShell** worked reliably.
- In PowerShell, avoid double quotes inside `git commit -m` strings — use a single-quoted
  here-string (`@'...'@`) with the closing `'@` at column 0.

---

## 3. Page structure (all inside `index.html`)

Order top to bottom: **Nav → Hero → KPIs → Latest updates → Current Work net → Future Work net →
Delivery summary → 2026 Roadmap → AI Strategy → (hidden AI Model view) → Footer**

- **Two neural nets** (HTML canvas, custom force-directed physics, no libraries):
  - **Current Work** — delivered / in-progress projects (solid dots)
  - **Future Work** — planned projects (hollow dashed dots)
  - Both are produced by one reusable `makeNet(cfg)` function, called twice.
- **Five pillars** (this mapping matters — used in code and in `UPDATES`):

  | Key | Pillar | Colour |
  |---|---|---|
  | `A` | Data Analytics & Power BI | `#E2231A` red |
  | `B` | Automations | `#0E8C6B` green |
  | `C` | Agents | `#E07B1A` orange |
  | `D` | **Apps** (renamed from "R&D") | `#6D5BD0` purple |
  | `E` | Tech Hold | `#2563EB` blue |

- **AI Operating Model view** — a second in-page view (`#view-aimodel`), hash-routed and currently
  **hidden**: its nav link is commented out. To re-enable, uncomment
  `<a class="lnk" href="#aimodel">AI Model</a>` in the nav.
- **Sub-hub support** — `groups[key].subgroups` can nest child dots under a parent node
  (ids use `::`, e.g. `B::Freight Department App`). Currently unused but kept for reuse.

### Key data objects in the `<script>`
- `UPDATES` — the notifications feed (see §4)
- `LINKS` — `"Project Name": "https://..."` — a project with a link gets a **↗ ring** and opens it
- `DESC` — `"Project Name": "tooltip text"`
- `groups` — the five pillars, each with `projects` (current) and `future` (planned) arrays

To **add a project**: add its name to the right `groups` array, plus a `DESC` entry, plus a
`LINKS` entry if it has a URL. To **remove**: delete from all three.

---

## 4. The `UPDATES` feed — rules that must be followed

One array drives **three** surfaces at once: the **Latest updates** cards, the **🔔 bell** panel,
and the **glowing dots** on the Current Work net.

```js
{date:"YYYY-MM-DD", type:"new"|"mod", net:"current", scope:"Apps",
 title:"...", note:"...", color:"#6D5BD0",
 url:"https://...",              // optional — card/bell opens it (shows ↗)
 nodes:["D:Exact Project Name"]}  // makes that dot glow
```

**Hard rules:**
1. **`date` = the ACTUAL current calendar date** when the entry is added. **Never copy the date
   from the entry above** — that bug shipped stale dates before. Check the real date first.
2. **Max 9 entries.** Add new at the **top**, delete the oldest at the **bottom** (FIFO).
3. `nodes` ids must **exactly** match a real Current Work node (`"<pillar letter>:<label>"`),
   or nothing glows.
4. Glow + feed are **current work only** — the Future Work net never glows (guarded in code).
5. `color` should match the pillar colour above.

---

## 5. Authentication (already working — don't rebuild it)

Sign-in is **client-side MSAL**, locked to the Sallaum tenant.

- **Entra app:** "PowerBI Refresh" — client ID `123e4020-6dce-489e-93a0-01a760276e81`
- **Tenant:** `0f0000f9-39cb-49a3-a562-ebe7dca0a3be`
- The **MSAL library is inlined inside `index.html`** (search `self-hosted, no CDN`) so the file
  is fully self-contained and immune to CDN/network blocking. This is why the file is ~450 KB.
- Enforced on **every non-local host**; skipped on `localhost` / `file://` so the file stays
  editable and previewable locally.
- Full-screen "Signing you in…" overlay; footer **Sign out** calls `__signOut()`.

**Adding a new host (custom domain, new Netlify site, etc.):** register the exact origin
(**no trailing slash**) as a **Single-page application** redirect URI on the PowerBI Refresh app
in Entra, or sign-in fails with `AADSTS50011`.

> **Known limitation:** this is a *soft* gate — MSAL controls who sees the rendered page, but the
> static HTML is still downloadable. It contains no secrets; every linked report/app has its own
> login. A true edge-level lock needs the SWA **Standard plan (~$9/mo)** plus a custom
> single-tenant Entra registration.

---

## 6. Design conventions

- Brand: navy `#0E3A6B`, red `#E2231A`. Font stack: Segoe UI / system sans.
- Favicon: inline SVG of the red Sallaum mark (two parallelograms).
- Naming: apps are titled **"<Department> App"** (e.g. Finance Department App, SMC App).
- Delivery cards show **Support hrs/mo** and **Est. saved hrs/mo**; the KPI strip headline is
  **450+ hrs/month saved** — keep these consistent if either changes.
- The logo `<img>` falls back to the "Sallaum Enterprises" text if `sallaum-logo.png` is absent
  (it is not in the repo — this fallback is intentional and fine).

---

## 7. Working agreements with the owner

- **Publish when asked** ("publish the dashboard" / "push the update") = commit + push to `main`.
- Prefer **commenting out** links rather than deleting them when something is temporarily hidden,
  so it is easy to restore.
- When adding an app, also add it to `UPDATES` so it glows and appears in the bell — unless told
  otherwise.
- Keep answers and changes tight; the owner iterates quickly in small, specific requests.

---

## 8. Quick orientation for a new session

```bash
git clone https://github.com/dataunitsallaum/Data-Unit.git
cd Data-Unit
# then just open index.html in a browser to preview (auth is skipped on file://)
```

Then read, in `index.html`: the `UPDATES` array, then `LINKS` / `DESC` / `groups`. That is where
essentially all content changes happen. The rest of the file is layout, canvas physics, and the
inlined MSAL library (skip it — it is vendor code).
