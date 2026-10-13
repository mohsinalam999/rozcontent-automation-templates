# Template 15 — Daily Business Digest

**The problem:** your business numbers live in four different sheets — leads in one, appointments in another, invoices in a third, orders in a fourth — and nobody opens all four before the day starts. So the overdue invoice, the 9 AM booking and yesterday's quiet sales day all ambush you later, one at a time.

**What this workflow does:** every morning at **07:30** it reads the same four sheets our other templates maintain and sends you **one** plain-text email: new leads from the last 24 hours, today's appointments, overdue invoices with the total outstanding, and yesterday's orders with revenue. If anything needs action — an overdue invoice, or appointments today — the subject line is flagged **🔔** so you can spot it half-asleep. Quiet sections say so honestly ("All quiet") instead of disappearing.

```
07:30 daily ─▶ CONFIG ─▶ Read Leads ─▶ Read Appointments ─▶ Read Invoices ─▶ Read Orders
                                                                                    │
                                                                             Build the Digest
                                                                                    │
                                                                          Anything urgent?
                                                                            ├─ Yes ─▶ 🔔 Digest — Action Needed
                                                                            └─ No ───▶ Digest — Daily
```

**Who it's for:** any owner running two or more of our templates (Lead Capture, Appointment Reminders, Invoice Chaser, E-commerce Order Assistant) — or anyone who keeps those four sheets by hand and wants them summarised before the day starts.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheets hold your numbers; Gmail sends the digest | Free |
| At least one of the four sheets below | The digest reads them — the more you run, the fuller it gets | Free (created by Templates 1, 4, 6 and 9, or by hand) |

## Which template creates which tab

Point each read node at the sheet that holds that tab. The four tabs can live in **one** spreadsheet or in four different ones — each read node has its own sheet-ID field.

| Read node | Tab name (exact) | Created by | Headers in row 1 (exact order) |
|---|---|---|---|
| Read Leads | `Leads` | Template 1 — Lead Capture & Follow-Up Machine | `Timestamp \| Name \| Email \| Company \| Interest \| Source \| Status` |
| Read Appointments | `Appointments` | Template 4 — Appointment Reminder & No-Show Reducer | `Client Name \| Email \| Service \| Appointment Date \| Appointment Time \| Status \| Reminder 24h Sent \| Reminder 2h Sent` |
| Read Invoices | `Invoices` | Template 6 — Invoice Chaser | `Invoice Number \| Client Name \| Email \| Amount \| Due Date \| Status \| Heads-Up Sent \| Due Reminder Sent \| Overdue 3 Sent \| Final Notice Sent` |
| Read Orders | `Orders` | Template 9 — E-commerce Order Assistant | `Timestamp \| Order ID \| Customer \| Email \| Total \| Currency \| Items \| Event` |

Don't run one of those templates yet? Two honest options: create that tab by hand with the headers above and keep it yourself, or leave that read node as it is — a missing tab is simply skipped (see *Good to know*).

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes left to right.
2. **Point the four reads at your sheets.** Open each Google Sheets read node (*Read Leads*, *Read Appointments*, *Read Invoices*, *Read Orders*) and paste the ID of the spreadsheet holding that tab, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`. The ID is the long string in the sheet's URL, between `/d/` and `/edit`. Tabs must be named exactly as in the table above.
3. **Connect your Google account.** Click each Google Sheets node (×4) and Gmail node (×2) and connect your account on each. You can reuse the same Google account; n8n treats Sheets and Gmail credentials separately.
4. **Edit the CONFIG node.**
   - `config_business_name` → your business name (shown at the top of the digest and in the subject)
   - `config_owner_email` → the inbox that gets the digest each morning
   - `config_currency` → the symbol shown before invoice amounts (default `$`; try `£`, `€`, `PKR `…)
5. **Set the workflow timezone.** Top-right ⋯ menu → **Settings** → set the timezone to your business timezone. The 07:30 start, "today", "yesterday" and the last-24-hours window all use it. Want a different start time? Open the *Daily — 07:30* node and change the hour/minute.
6. **Test it by executing manually.** Add a few rows you can check by eye: a lead with a Timestamp from a couple of hours ago in `Leads`, an appointment dated **today** in `Appointments`, an unpaid invoice with a `Due Date` last week in `Invoices`, and an order Timestamped **yesterday** in `Orders`. Then click **Execute Workflow**. Check: one digest email arrived, the counts and totals match what you just typed, and — because there is an overdue invoice and an appointment today — the subject is the 🔔 "Action needed" one. (There's no separate test trigger node: with the workflow open, Execute Workflow runs the whole chain once, schedule or not.)
7. **Check the first few real digests by hand.** For the first few mornings, open the sheets alongside the digest and confirm each section. A tab that fails — wrong name, wrong sheet ID — is skipped silently, so this early check is how you catch a typo before you start trusting the numbers.
8. **Go live.** Flip the workflow to **Active**. From then on it runs itself at 07:30 every day.

## Credentials used

- **Google Sheets OAuth2** — read the Leads, Appointments, Invoices and Orders tabs (×4 nodes, read-only in practice — this workflow never writes to a sheet)
- **Gmail OAuth2** — send the digest, flagged or normal (×2 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **How the four reads combine:** Google Sheets "get" nodes *replace* the items flowing through a chain rather than merging them, so the four reads run in sequence and the single *Build the Digest* Code node pulls each one back by name — `$('Read Leads').all()`, `$('Read Appointments').all()`, and so on. If you rename a read node, update its name in that Code node too.
- **Any sheet you don't use yet is simply skipped.** All four reads are set to continue on error, with the error output left unconnected, and the Code node treats a failed read as zero rows. A business not running Template 9 yet still gets its leads, appointments and invoices every morning.
- **The urgent rule is deliberately simple:** the 🔔 subject fires when there is at least one overdue invoice **or** at least one appointment today. New leads and orders, however good, never flag a digest — they're morning reading, not action items.
- **What counts as what:** a lead is "new" if its `Timestamp` falls in the last 24 hours; an appointment counts if its `Appointment Date` is today and its Status isn't `Cancelled`; an invoice is overdue if it isn't marked `Paid` (Cancelled / Written off are also left out) and its `Due Date` is before today; orders are counted by their `Timestamp` falling yesterday. Dates are read the way the other templates write them — `YYYY-MM-DD` for appointment dates and due dates.

## Honest limits

- **The digest is only as good as the sheets behind it — garbage in, garbage out.** A lead with no Timestamp, an invoice nobody marked Paid, an appointment typed as `20/10/2026` instead of `2026-10-20`: the digest faithfully summarises the sheet, errors included. It cannot clean your data for you.
- **The money figures are simple sums of what's typed in the sheets — NOT accounting.** No partial payments, no refunds, no tax, no credit notes. Mark an invoice Paid the day the money lands, or it will keep showing as outstanding.
- **Mixed currencies are summed per currency symbol, exactly as typed** — order revenue appears as e.g. `$50 + £30`, never converted into one number. Invoice totals use your single `config_currency` symbol, because the Invoices sheet has no currency column of its own.
- **A missing tab is skipped silently.** That is a feature when you genuinely don't use a sheet yet — and a trap when a tab name is typo'd or a sheet ID is wrong, because the digest will just report that section as quiet. Step 7 (checking the first few digests by hand) exists for exactly this reason.
- **One email, once a day, at 07:30.** If n8n is down at 07:30, that day's digest is missed, not queued — the next morning's digest still covers its own 24-hour/yesterday windows, so a missed day leaves a gap rather than a double send.

## Want the grown-up version?

The RozContent paid build adds: the same digest on **WhatsApp** so it lands where you actually look first, week-on-week comparisons ("leads up 3 on last week") instead of single-day snapshots, a combined dashboard sheet kept up to date automatically, and monitoring so a broken sheet connection is flagged to us — not silently skipped — before your morning numbers go missing.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
