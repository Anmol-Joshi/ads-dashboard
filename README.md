# Sales Dashboard — Deploy to Vercel

A standalone, single-file dashboard for tracking book sales, ad spend, and net profit. No backend, no database, no signup. Data lives in each visitor's browser. Anyone with Claude + Meta Ads MCP can push data straight in via a URL.

## Files

```
dashboard/
├── index.html    ← The whole app (HTML + CSS + JS, single file)
└── README.md     ← This file
```

## Deploy in 60 seconds

### Drag & drop (no terminal)

1. Go to **vercel.com/new**, sign in (free works).
2. Drag the `dashboard/` folder into the upload area, or zip it first if your browser prefers.
3. Click **Deploy**. ~20 seconds later you get a URL like `dashboard-xyz.vercel.app`.

### CLI

```bash
cd dashboard
npx vercel deploy --prod
```

### Anywhere else

It's plain static HTML — works on Netlify Drop, GitHub Pages, Cloudflare Pages, your own server, or just opened from your computer (offline-capable).

## Three ways data gets in

### 1. Manual entry / CSV paste (works today, no setup)

- **+ Sale** and **+ Spend** buttons for one-off entries
- **⇣ Import** for bulk paste from SuperProfile CSV (sales) or Meta Ads Manager CSV (spend)

### 2. Claude with Meta Ads MCP — one-click URL

Anyone with **Meta Ads MCP** enabled in Claude can fetch their data and push it into your dashboard via a merge URL. The flow:

1. User opens Claude (claude.ai or the app)
2. User pastes a prompt: *"My dashboard is at `https://...vercel.app`. Use Meta Ads MCP to fetch my last 30 days of ad spend for account `XXXX`, then give me a merge URL."*
3. Claude calls Meta MCP, builds a JSON payload, base64-encodes it, returns the final URL
4. User clicks the URL → dashboard opens, asks "Merge this data?" → click Yes → done

Inside the dashboard, click **⇪ Share → Use with Claude** to copy the exact prompt with your URL pre-filled.

The merge URL format is:

```
https://ads-dashboard-lilac.vercel.app/#merge=<BASE64_ENCODED_JSON>
```

Where the JSON is:

```json
{
  "spend": [
    {"date": "2026-04-27", "amount": 450.32, "platform": "meta", "campaign": "Modern Mans Edge"},
    {"date": "2026-04-28", "amount": 520.10, "platform": "meta", "campaign": "Modern Mans Edge"}
  ],
  "sales": [
    {"date": "2026-04-27", "qty": 1, "price": 999, "source": "meta", "note": "..."}
  ]
}
```

`sales` and `spend` are both optional. The dashboard merges incoming data without wiping existing records. Spend records that share a date with an existing entry are replaced (so re-pulling overwrites old numbers cleanly).

**Limits:** URL hash carries ~2KB of data comfortably (≈60 days of daily spend). Past that, browsers and messaging apps may truncate. For larger imports, use option 3.

### 3. Claude with Meta Ads MCP — CSV paste (no size limit)

If the merge URL would be too big, ask Claude to print a CSV instead and paste it into the dashboard's Meta CSV importer. Same **Use with Claude** tab in the Share modal has this prompt ready to copy.

## Sharing your dashboard with someone else

Inside the **⇪ Share** button:

| Method | When to use |
|---|---|
| **Spreadsheets** | Sending a report. Excel (.xlsx) with Daily Summary + Sales + Spend + lifetime Snapshot tabs. CSV files for individual ledgers. |
| **Use with Claude** | Letting someone push their Meta data into the dashboard via Claude. |
| **Share URL** | Quick — generates a link with your data baked in. Anyone who opens it sees your snapshot. Long URLs may break in some apps. |
| **JSON** | Reliable backup or hand-off. Recipient pastes into Import tab. |
| **Print → PDF** | One-pager client report. `Ctrl+P` / `Cmd+P`. Print styles are built in. |

## Why this architecture

This is a **single-tenant tool that scales via Claude as the data layer**:

- Each user (or each client of yours) deploys their own copy. Free Vercel tier handles this fine — unlimited static sites.
- Meta data integration happens via Claude + MCP, not via a backend you have to build/maintain.
- No auth, no database, no OAuth, no Meta app review.
- Data privacy is automatic — each browser holds only its own data.

For agency use: deploy one dashboard per client (`{client}.vercel.app`), use Claude weekly to pull their Meta data into each, and send the URL. Clients view live numbers; you don't manage shared infrastructure.

## When to graduate to a real SaaS

If you ever need:

- Many users logging in to one dashboard
- Live OAuth into Meta (no Claude required)
- A team viewing the same data simultaneously
- Webhooks, scheduled jobs, email alerts

…that's a real build — roughly 2–4 weeks of engineering with Next.js + Supabase. Treat this dashboard as the validated UX prototype — run it for 4–6 weeks, see what people actually need, then build the SaaS.

## Configuring the dashboard

No code edits needed. Use the **⚙ Settings** button:

- Brand / product name
- Default price
- Platform fee % (SuperProfile is ~5%, Razorpay direct is ~2%)
- Erase all data (with confirmation)
