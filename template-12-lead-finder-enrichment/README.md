# Template 12 — Lead Finder & Enrichment

**The problem:** you need a list of local businesses to pitch — so you either spend hours copy-pasting names from a map, or you buy a stale list of strangers from a data broker and feel grubby about it. Both waste the one thing outreach can't spare: your time and your credibility.

**What this workflow does:** you type in a niche and a city, press run, and it searches **public OpenStreetMap business listings**, cleans and dedupes them, and — where a listing includes a website — visits that business's **own website** to pick up the contact details published there. Every prospect lands as a tidy row in a Google Sheet, and you get an email when the list is ready. This is RozContent's own outreach method, productised: public data, found fast, worked personally.

```
Run manually ─▶ CONFIG (niche + city) ─▶ Search public listings (OpenStreetMap)
                                                ├─ error ─▶ Alert you
                                                └─▶ Normalise & dedupe ─▶ Has a website?
                                                                                ├─ Yes ─▶ Fetch their website ─▶ Extract published email/phone ─▶ Save to Sheet
                                                                                ├─ Yes, but site fails ────────────────────────────────────────▶ Save to Sheet (no contact details)
                                                                                └─ No ─────────────────────────────────────────────────────────▶ Save to Sheet
                                                                                                                                                              └─▶ Notify you — list ready
```

**Who it's for:** agencies, freelancers and local-service businesses building a prospect list for personalised outreach — web designers hunting cafés with no site, marketers listing local clinics, anyone whose pipeline starts with "who could we help in this town?"

**Setup time:** ~20 minutes · **Difficulty:** Easy–Medium

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet is your prospect list; Gmail sends the alerts | Free |
| OpenStreetMap / Nominatim search | The public listings source — no account, no API key | Free, with a fair-use policy (see *Good to know*) |
| (Optional upgrade) A Google Places API key | Richer results — website and phone on most rows | Paid, your own Google billing; see the upgrade note below |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **one tab**, named exactly:
   - `Prospects` with headers in row 1: `Timestamp | Business | Category | Address | Website | Email | Phone | Source | Status`
   
   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into the Google Sheets node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the two Gmail nodes (you can reuse the same Google account; n8n treats Sheets and Gmail credentials separately).
4. **Edit the CONFIG node.** Set:
   - `config_niche` → the business type you want (e.g. `dentist`, `coffee shop`, `plumber`)
   - `config_city` → the area to search (e.g. `Austin, TX`)
   - `config_owner_email` → the inbox that should receive the alerts
   - `config_max_results` → keep it small — `20` is the default and the polite maximum for a single run
   - `config_business_name` → your business name (used in the notification email)
5. **Put a real contact in the User-Agent.** Open *Search Public Listings* and replace `owner@yourbusiness.com` in the User-Agent header with your own email. OpenStreetMap's usage policy asks automated users to identify themselves with a valid contact — don't skip this.
6. **Test it.** With the workflow open, click **Execute workflow** on the *Run Lead Search Manually* trigger. Check: rows appear in the `Prospects` tab, and a "list ready" email lands in the owner inbox with the count. Try a niche/city you know well so you can sanity-check the businesses it finds.
7. **Run it for real.** Change the CONFIG niche/city whenever you want a fresh list and press run again. There is deliberately **no schedule and no Active toggle needed** — this workflow is manual-trigger by design, to stay inside OpenStreetMap's low-volume usage policy. Before re-running the same search, tidy or clear earlier rows if you don't want repeats in the sheet (the workflow dedupes within a run, not against previous runs).

## Credentials used

- **Google Sheets OAuth2** — append prospect rows (×1 node)
- **Gmail OAuth2** — send the list-ready notification and the search-failed alert (×2 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect. The OpenStreetMap search needs no credential at all.

## Good to know

- **PUBLIC DATA ONLY — this is a hard line, not fine print.** This template reads openly published business listings and contact details printed on a business's own website. It does not scrape behind logins, does not buy data, does not guess or construct email addresses, and does not touch personal (non-business) data. What you do with the list is on you: outreach laws (CAN-SPAM in the US, GDPR/PECR in the UK/EU, and equivalents elsewhere) are the user's responsibility. One personalised email to a business you can genuinely help beats 100 blasts — and keeps you legal and un-blocked.
- **Be honest with yourself about the free data.** Nominatim search does **not** return contact tags (website/phone) by default, and most OpenStreetMap listings simply don't carry them. So in practice this free version mostly produces **name / category / address** rows — genuinely useful for mapping a market, visiting in person, or looking businesses up one by one — and the enrichment branch only fires for the minority of listings that *do* include a website field. Expect many rows with no email. That is normal, not a bug.
- **OpenStreetMap / Nominatim usage policy.** Their policy asks for low volume: at most ~1 request per second and no bulk harvesting of businesses at scale. This template is deliberately manual-trigger, one search per run, small batches (default 20). Don't loop it, schedule it hourly, or point it at a whole country — that's how free community services get shut down for everyone.
- **Coverage varies wildly.** OpenStreetMap is community-mapped: a trendy city district may have every café listed, while a small town may have almost nothing, and category names depend on how volunteers tagged things. If a search comes back thin, try a broader niche word (`restaurant` instead of `bistro`) before assuming the workflow is broken.
- **The phone regex is best-effort.** It lifts the first phone-shaped number published on the homepage. That is usually the main business line — but it can occasionally catch a booking or delivery number printed nearby. Glance before you dial.
- **Upgrade path — Google Places (better data, your key, your bill).** If you need website and phone on most rows, swap the *Search Public Listings* node for a Google Places Text Search call using **your own** API key: `GET https://maps.googleapis.com/maps/api/place/textsearch/json?query=<niche>+in+<city>&key=YOUR_KEY`, then a Place Details call per result. Places returns `website` and `formatted_phone_number` (plus `name`, `formatted_address` and `types`) — feed those into the *Normalise & Dedupe Listings* node in place of the Nominatim fields and the rest of the workflow works unchanged; far more listings will then take the enrichment branch. Places is a **paid** Google service (it has a monthly free usage tier that changes from time to time — check Google's current pricing before you rely on it), and the key and the bill are yours. This template ships without it on purpose: the free version should cost nothing to try.

## Want the grown-up version?

The RozContent paid build adds: Google Places enrichment wired in (website + phone on most rows), email verification before a row is marked outreach-ready, dedupe against your master prospect list so you never pitch the same business twice, a weekly scheduled search that only adds genuinely new businesses, and a first-touch personalisation draft per prospect — still sent by a human, one at a time.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
