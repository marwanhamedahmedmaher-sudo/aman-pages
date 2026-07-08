# aman-pages

Static, browser-facing web tools for **Aman** supervisors, published via **GitHub Pages**.

These are the pages a supervisor actually opens in a browser to export field-visit
reports and manage rep accounts. They are **plain HTML/JS** — no build step. GitHub
Pages rebuilds automatically on every push to `main`.

> ⚠️ **This repo is a _publish target_, not the source of truth.**
> The canonical copies live in the app repo **`Aman_One`** under `docs/`. Edit there,
> then copy the file here (see [Updating a page](#updating-a-page)). Never let these two
> drift — a change made only here will be lost the next time someone syncs from `Aman_One`.

---

## Pages

| File | Purpose | URL |
|------|---------|-----|
| `index.html` | **Merged supervisor portal — the default landing page.** Visit-export + rep-management in one page with two tabs («تقارير الزيارات» + «إدارة المناديب»). Byte-identical to `supervisor-portal.html`. | https://marwanhamedahmedmaher-sudo.github.io/aman-pages/ |
| `supervisor-portal.html` | Same merged portal, kept at its original URL so existing links/bookmarks keep working. | https://marwanhamedahmedmaher-sudo.github.io/aman-pages/supervisor-portal.html |
| `admin-portal.html` | **Rep-management portal** (standalone) — list / create / suspend / reactivate / reset-password for sales reps. | https://marwanhamedahmedmaher-sudo.github.io/aman-pages/admin-portal.html |
| `visit-export.html` | **Visit-export tool** (standalone) — supervisor logs in, picks a from/to date range, downloads an Excel workbook of field visits (photos embedded). Was previously the root page. | https://marwanhamedahmedmaher-sudo.github.io/aman-pages/visit-export.html |

All pages are RTL Arabic.

---

## How it works

Each page is a thin client. It:

1. Logs the user in against **Supabase Auth** (phone + password) using the **public anon key**.
2. Reads the caller's own `name` / `role` / `business_unit` row to show an identity banner
   and **refuse non-supervisor/admin logins** (guards against browser autofill silently
   logging in as a rep → empty/denied result).
3. Calls a **Supabase Edge Function** for the privileged work:
   - `export-visits-xlsx` — builds the Excel export.
   - `admin-users` — the rep-management actions.

**All real authentication and authorization happens server-side in those edge functions**,
which live in `Aman_One` and run on the prod Supabase project. In particular:

- A **supervisor is scoped to their own `business_unit`** — enforced in the edge functions,
  not here. The pages only *display* the scope; they cannot widen it.
- The anon key embedded in these pages is **public by design** (that's what the anon key is
  for). It grants nothing beyond what Supabase RLS + the edge-function role gates already allow.

### Why GitHub Pages instead of serving from Supabase?

Supabase now serves edge-function HTML on `*.supabase.co` as `Content-Type: text/plain`
(anti-phishing enforcement), which breaks these pages in a browser. GitHub Pages serves
them as real HTML. The old `visit-export-page` / `admin-portal-page` edge functions in
`Aman_One` are therefore **dead paths** — parked, not used.

Because Pages can't send `X-Frame-Options`, each page carries a `noindex` meta tag and a
small JS **frame-buster** to refuse being embedded in an iframe.

---

## Updating a page

The source of truth is **`Aman_One/docs/`**. To change a page:

1. Edit the canonical file in `Aman_One` (`docs/visit-export.html`,
   `docs/admin-portal.html`, or `docs/supervisor-portal.html`) and open a PR there so the
   change is reviewed alongside any edge-function changes it depends on.
2. Copy the updated file into this repo. **Mapping:**
   - `docs/supervisor-portal.html` → copy to **both** `supervisor-portal.html` **and** `index.html` (the root page mirrors the merged portal).
   - `docs/visit-export.html` → copy to `visit-export.html`.
   - `docs/admin-portal.html` → copy to `admin-portal.html`.
3. Commit + push to `main`. GitHub Pages redeploys automatically (usually within a minute).

If you changed anything the page calls, remember the **backend logic lives in the
`Aman_One` edge functions** and must be deployed to Supabase separately — updating the HTML
here does **not** change behavior on its own.

---

## Repo settings

- **Visibility:** public (required for GitHub Pages on this plan).
- **No secrets** live here — only the public Supabase URL + anon key. Never commit a
  service-role key, rep roster, or any merchant/PII export to this repo.
