# Template 1 — Lead Capture & Follow-Up Machine

**The problem:** a prospect fills in your website form… and hears back tomorrow, or never. Most small-business leads go cold in the first few hours — not because the offer is wrong, but because the follow-up is manual.

**What this workflow does:** the moment a lead arrives, it is saved to a Google Sheet, you are notified, the lead gets a polite thank-you email immediately, and a gentle nudge two days later if the conversation hasn't started. You just show up and reply like a human.

```
Website form ─▶ Webhook ─▶ Clean up ─▶ Valid email?
                                            ├─ Yes ─▶ Save to Sheet ─▶ (error? → alert you)
                                            ├─ Yes ─▶ Day 0 thank-you email ─▶ Wait 2 days ─▶ Day 2 nudge ─▶ Log it
                                            ├─ Yes ─▶ Notify you instantly
                                            └─ No ───▶ Flag it to you, send nothing
```

**Who it's for:** any business collecting enquiries from its website — clinics, agencies, trades, coaches, local services.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet is your mini-CRM; Gmail sends the emails | Free |
| A website form that can POST JSON to a URL (or a form tool like Tally/Typeform with a webhook) | Delivers the lead | — |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Leads` with headers in row 1: `Timestamp | Name | Email | Company | Interest | Source | Status`
   - `Follow-up Log` with headers: `Timestamp | Email | Stage | Detail`
   
   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail nodes (you can reuse the same Google account; n8n treats Sheets and Gmail credentials separately).
4. **Edit the CONFIG node.** Set:
   - `config_owner_email` → the inbox that should get new-lead alerts
   - `config_business_name` → your business name (used in the email wording)
5. **Read the two emails and make them yours.** Open *Day 0 — Thank-You Email* and *Day 2 — Nudge Email* and adjust the tone. Keep them short and human — that's the point.
6. **Point your form at the webhook.** Open the *Lead Form Webhook* node → copy the **Production URL** → set your form to POST these JSON fields to it: `name`, `email`, `company`, `service_interest`, `source`. (No form yet? Test first with step 7, then wire the form.)
7. **Test it.** With the workflow open, click **Execute workflow**, then send a test lead — e.g. with curl:
   ```bash
   curl -X POST https://YOUR-N8N-URL/webhook-test/new-lead \
     -H "Content-Type: application/json" \
     -d '{"name":"Test Person","email":"you@youremail.com","company":"Test Co","service_interest":"Website build","source":"Manual test"}'
   ```
   Use your own email as the test lead so you can see the Day 0 email arrive. Check: a row in `Leads`, an alert in the owner inbox, a thank-you in the lead inbox.
8. **Go live.** Flip the workflow to **Active** (top-right toggle), then swap the form to the **Production URL** (`/webhook/new-lead`, without `-test`).

## Credentials used

- **Google Sheets OAuth2** — append rows to your sheet (×2 nodes)
- **Gmail OAuth2** — send the thank-you, nudge, owner alert and error alert (×4 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- The Day 2 nudge uses a **Wait** node: the execution stays open for two days. Fine at normal lead volumes; don't be alarmed by "running" executions in the list.
- This free version does not detect replies. If a lead replies on Day 1 and you are mid-conversation, the Day 2 nudge can still fire — the wording is deliberately soft ("floating this back up"), but check the production note in the workflow if that matters to you.
- Duplicate submissions create duplicate rows. A weekly 2-minute tidy of the sheet is enough at this scale.

## Want the grown-up version?

The RozContent paid build adds: reply detection (the sequence stops the moment a lead answers), a Day 5 final touch, WhatsApp alerts instead of email, updating the master lead row instead of a separate log, and monitoring so a broken step gets fixed before it costs you a client.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
