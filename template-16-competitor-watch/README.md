# Template 16 — Competitor Price & Page Watch

**The problem:** your competitors change their prices, plans and offers on their own websites — quietly, and usually without announcing it. You find out weeks later, from a customer who already bought from them. Checking a handful of rival pages by hand every day sounds simple, until the day you skip it — which is most days.

**What this workflow does:** once a day it fetches every competitor page on your Watch List, strips each page down to its visible text, hashes that text, and compares the hash with the one stored from the last check. If the text changed, you get an email alert. Either way, it writes the new hash and check time back to the sheet, so the baseline stays fresh and you are never alerted twice about the same change. The first run only sets baselines — no alerts.

```
Daily 08:15 (or Run Manually) ─▶ CONFIG ─▶ Read Watch List ─▶ Keep Watchable Rows ─▶ Fetch Page
                                                                                          ├─ Error ─▶ Alert — Page Unreachable
                                                                                          └─ OK ────▶ Extract & Compare ─▶ Page Changed?
                                                                                                                                                              ├─ Yes ─▶ Alert Owner — Competitor Changed Something + Update Watch List
                                                                                                                                                              └─ No ───▶ Update Watch List (new hash, Last Checked, note)
```

**Who it's for:** e-commerce stores tracking rival pricing, agencies watching client competitors, real-estate businesses tracking rival listings and offers — any business that needs to know when a competitor's pricing, plans or offers move.

**Setup time:** ~20 minutes · **Difficulty:** Easy–Medium

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Watch List lives in Google Sheets; Gmail sends the alerts | Free |
| 2–3 competitor page URLs to start with | The pages this workflow checks every day — pricing and offers pages work best | Free |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes in order.
2. **Create the Google Sheet.** Make a new sheet (any name) with **one tab** named exactly `Watch List` and headers in row 1:
   `Label | URL | Last Hash | Last Checked | Last Change Note`
   
   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes — *Read Watch List* and *Update Watch List* — replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Add your watch list.** Add one row per page, starting with 2–3 competitor pages: a friendly `Label` (e.g. `Rival Co — Pricing`) and the full `URL`. Leave `Last Hash`, `Last Checked` and `Last Change Note` empty — the workflow fills those in. Rows without a URL starting with `http` are skipped silently by *Keep Watchable Rows*.
4. **Connect your Google account.** Click *Read Watch List* → credential dropdown → **Create new** → sign in with your Google account. Repeat on *Update Watch List* and on the two Gmail nodes (*Alert — Page Unreachable* and *Alert Owner — Competitor Changed Something*). You can reuse the same Google account; n8n treats Sheets and Gmail credentials separately.
5. **Edit the CONFIG node.** Open *CONFIG — Edit Me* and set:
   - `config_owner_email` → the inbox that should receive change and unreachable-page alerts
   - `config_business_name` → your business name
6. **First run — baselines only.** Click **Execute workflow** (the *Run Manually* trigger runs the same flow as the schedule). Expect **no alert emails**. Check the sheet: every watched row should now have a `Last Hash`, a `Last Checked` timestamp, and a `Last Change Note` like `Baseline set 2026-10-13`. Run it manually a second time — with nothing changed, you should still get no alert, and the notes should read `No change`.
7. **Test the alert.** In the sheet, edit one row's `Last Hash` cell to a wrong value (e.g. `deadbeef`), then run the workflow manually again. You should get exactly one email — *👀 Competitor changed something: [that page's Label]* — with the page URL and a short sample of the page's visible text as fetched that day. Check the sheet afterwards: that row's hash has been written back to the correct value and its note reads `Changed …`, so the alert will not repeat tomorrow.
8. **Go live.** Flip the workflow to **Active** (top-right toggle). The *Daily — 08:15* trigger will now run it once a day. Add more pages to the Watch List whenever you like — new rows get their baseline silently on the next run.

## Credentials used

- **Google Sheets OAuth2** — read the Watch List and update it with the new hash, check time and note (×2 nodes)
- **Gmail OAuth2** — send the change alert and the unreachable-page alert (×2 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **The hash detects THAT the visible text changed — not WHAT changed, and not the new price.** This workflow does not extract or compare actual prices. A changed headline, a new offer, a removed plan and a rotated banner all look the same to a hash; the alert email includes a short text sample, but you still need to open the page to see what moved. A real price-diff engine is part of the paid build.
- **Banners, dates and rotating content cause occasional false alarms.** Anything that changes the visible text — a countdown, a stock counter, a daily banner — changes the hash. Watch stable pages like pricing and offers pages, not busy homepages.
- **JavaScript-heavy pages may fetch as near-empty shells.** *Fetch Page* reads the page's raw HTML, not a rendered browser view, so a page built entirely in JavaScript can come back almost empty and look "unchanged" forever.
- **One polite fetch per page per day.** The workflow checks each listed page once a day — keep it that way, and respect each site's robots.txt and terms.
- **Some sites block bots — and that's their choice.** If a fetch fails, *Alert — Page Unreachable* emails you, and that page's baseline hash is left untouched (it was not compared that day). If the same page fails every day, the site is probably blocking automated fetches; remove it from the Watch List rather than trying to work around the block.
- **First run never alerts.** Empty `Last Hash` cells are filled in as baselines, silently. Change alerts only start from the second run onwards.

## Want the grown-up version?

The RozContent paid build adds: real price extraction and price-diff alerts that tell you the old price, the new price and the difference (not just "something changed"), rendered-browser fetching for JavaScript-heavy pages, a full change-history log so you can see when a rival moved and by how much, and WhatsApp or Slack alerts instead of email.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
