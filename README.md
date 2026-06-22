# Sallaum Enterprises — Data & AI Dashboard · Azure Static Web Apps deployment ($0/month)

Hosts the single-file dashboard (`index.html`) on **Azure Static Web Apps (Free plan)**, **auto-deployed from GitHub**.

**Cost: $0/month.** No build step, no Functions, no database, no auth — it's a static, read-only page.

**Access: public.** Anyone with the link can view the dashboard. It contains no secrets; the live Power BI reports it links to still require their own Microsoft login. (If you later want the whole site gated to Sallaum accounts, that needs the SWA Standard plan, ~$9/mo — ask and I'll add it.)

---

## What's in this folder (this is your GitHub repo)
- `index.html` — the dashboard (refresh this copy before each deploy — see "Updating" below)
- `staticwebapp.config.json` — routing + security headers
- `.gitignore`
- *(the GitHub Actions deploy workflow is created automatically by Azure when you link the repo in Step 2)*

---

## One-time setup (~10 min, all in your accounts)

### 1 — Push to GitHub
Create a repo (public or private) and push this folder's contents (root = `index.html`, `staticwebapp.config.json`).

```bash
cd sallaum-dashboard-azure
git init
git add .
git commit -m "Sallaum Data & AI dashboard"
git branch -M main
git remote add origin https://github.com/<you>/<repo>.git
git push -u origin main
```

### 2 — Create the Static Web App (Free plan)
Azure Portal → **Create a resource** → **Static Web App**:
- **Plan type: Free**
- **Source: GitHub** → authorize → pick your repo + `main`
- **Build presets: Custom**
  - **App location:** `/`
  - **Api location:** *(leave blank)*
  - **Output location:** *(leave blank)*
- **Create.** Azure adds the deploy workflow + the `AZURE_STATIC_WEB_APPS_API_TOKEN` secret to your repo and runs the first deploy (~1–2 min).
- Note the URL: `https://<name>.azurestaticapps.net`.

### 3 — Done
Open the URL — the dashboard loads. Share that link with management.

---

## Updating the dashboard later
The canonical file is `C:\Users\wael.frashy\Downloads\index.html`. To publish changes:

```bash
copy "C:\Users\wael.frashy\Downloads\index.html" "C:\Users\wael.frashy\Downloads\sallaum-dashboard-azure\index.html"
cd sallaum-dashboard-azure
git add index.html
git commit -m "Update dashboard"
git push
```

→ Azure rebuilds and goes live in ~1 minute.

---

## Notes
- **Logo:** the page references `sallaum-logo.png`, which isn't included — so the nav shows the "Sallaum Enterprises" text fallback (intended behavior). To show an image instead, drop a `sallaum-logo.png` into this folder before pushing.
- **`logisoft-roadmap.html` link:** the "Belgium Customs Automation" dot links to `logisoft-roadmap.html`. That file isn't in this folder, so the link will 404 until you either add the file here or remove the link in `index.html`.
- **Custom domain (optional):** Static Web App → **Custom domains** → add e.g. `dataunit.sallaum.com` and follow the DNS instructions (free, included).
