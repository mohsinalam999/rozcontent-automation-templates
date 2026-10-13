# Template 6 — Invoice Chaser (Polite Payment Reminders)

**The problem:** late payment is rarely malice — it's your invoice sitting unread in a busy inbox while theirs gets paid because someone chased it. Chasing by hand is awkward, so it happens late, all at once, and crosser than it needed to be. A ladder of small, polite, *on-time* reminders gets invoices paid without burning the relationship.

**What this workflow does:** every morning it reads your invoices sheet and sends whichever stage each unpaid invoice has reached — a friendly heads-up **3 days before** the due date, a reminder **on the day**, a firmer note at **3 days overdue**, and a final notice at **7 days**. Each stage fires **once**, every send is logged, and the moment a row is marked **Paid** the chasing stops. On Mondays you get one summary of everything outstanding.

```
Daily ─▶ Read Invoices ─▶ Plan ─┬─ Stage reached? ─▶ Chaser email (right tone for the stage) ─▶ Stamp ─▶ Log
                                 └─ Monday? ────────▶ Outstanding summary ─▶ To you
```

**Who it's for:** freelancers, studios, agencies, trades — anyone who sends invoices and waits too long to chase them.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet is your invoice register; Gmail sends the reminders | Free |
| One line of payment instructions | Bank details or a payment link, pasted into every reminder so paying is effortless | — |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes.
2. **Create the sheet.** One Google Sheet with **two tabs**, named exactly:
   - `Invoices` with headers in row 1: `Invoice Number | Client Name | Email | Amount | Due Date | Status | Heads-Up Sent | Due Reminder Sent | Overdue 3 Sent | Final Notice Sent`
   - `Chase Log` with headers: `Timestamp | Invoice Number | Email | Stage | Detail`
   
   Copy the sheet's ID from its URL and paste it into all three Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account** on the Sheets nodes (×3) and Gmail nodes (×2).
4. **Edit the CONFIG node.**
   - `config_owner_email` → where the Monday summary goes
   - `config_business_name` → signs every reminder
   - `config_currency` → symbol shown before amounts (default `$`; try `£`, `€`, `PKR `…)
   - `config_payment_instructions` → exactly how to pay you. This line appears in **every** reminder — make it copy-paste easy (account name + number, or your payment link)
5. **Enter invoices the way the workflow expects.** One row per invoice: `Amount` as a plain number (`1250`, not `$1,250 — thank you`), `Due Date` as `YYYY-MM-DD`, `Status` blank or `Unpaid`/`Sent` while open. The four `…Sent` columns stay empty — the workflow fills them.
6. **Read the four tones.** The email wording lives in the *Plan Invoice Chases* node, in plain English, one block per stage. Read them before going live and soften or firm up the wording to match how *you* talk to clients — that voice is the whole product.
7. **Test it.** Add four fake invoices addressed to **your own** email: one due in 3 days, one due today, one due 5 days ago, one due 9 days ago — plus a fifth due 9 days ago with Status `Paid`. Click **Run Manually**. Check: four chaser emails arrived (one per stage, none for the Paid one), each row got exactly one new stamp, and `Chase Log` has four lines. Row 3 is the honest test of tone — would you be annoyed receiving it? Adjust until not.
8. **Go live.** Flip the workflow to **Active**. Then keep the one habit: **when money lands, mark the row Paid.** That word is the off-switch.

## Credentials used

- **Google Sheets OAuth2** — read invoices, stamp stages, append the log (×3 nodes)
- **Gmail OAuth2** — the chaser emails and the Monday summary (×2 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **How the "stamp once" trick works:** the *Mark Stage as Sent* node rewrites all four stamp columns in one update — the stage that just fired gets today's timestamp, the other three are written back exactly as they were. That is what guarantees no stage ever sends twice, without holding anything in memory.
- **The stages watch windows, not moments** (except the due-date reminder): heads-up fires on the first daily run 1–3 days before due, the firmer note anywhere in days 3–6 overdue, the final notice from day 7 on. If n8n is down for a day, the overdue ladder still catches up — only a missed due-*day* reminder is skipped.
- **The Monday summary counts what the sheet shows** as unpaid: anything not marked Paid, Cancelled or Written off. If the total looks wrong, a row somewhere needs its Status updated — the summary is a useful lie-detector for your own bookkeeping.
- An invoice entered *already overdue* (say you add a 10-day-old invoice today) starts at the matching rung — it gets the final notice, not the friendly heads-up. Sensible, but worth knowing before you backfill old invoices.

## Honest limits

The chaser knows only what the sheet knows: a client who paid yesterday still gets chased today if nobody marked the row Paid — keep the sheet current, or have the paid version connected to your accounting tool so payment flips the switch itself. It is email-only, from your Gmail; it adds no late fees, pauses nobody's work and starts no collections process. The final notice *says* work may pause — whether it does is your call, and it should be.

## Want chasing that stops itself the second money lands?

The RozContent paid build adds: connection to your invoicing/accounting tool so Paid status updates itself, card-payment links inside every reminder (one click beats bank details), automatic invoice creation and numbering, and a live "who owes what, since when" view for your Monday coffee.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
