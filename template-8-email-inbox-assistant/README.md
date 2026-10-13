# Template 8 — Email Inbox Assistant

**The problem:** your inbox is a mix of real enquiries, existing customers needing help, invoices, and noise — and the genuine lead sits unread for six hours because it arrived between two newsletters. Reading everything yourself doesn't scale, and ignoring it costs sales.

**What this workflow does:** every new email is classified by AI as a **lead**, **support**, **billing** or **spam-or-other**, given an urgency flag and a 2-sentence summary, and logged to a Google Sheet. Leads trigger an instant alert to you — summary plus a ready-to-send draft reply included. Support emails get their draft sent to **you only**, for review. Nothing is ever auto-sent to a customer.

```
New email ─▶ Gmail Trigger (polls every minute) ─▶ Clean up ─▶ CONFIG ─▶ AI: classify & draft
                                                                             ├─ error? ─▶ Alert you (AI step failed)
                                                                             └─ ok ─▶ Parse ─▶ Log to Inbox Log sheet
                                                                                                  ├─ lead? ────▶ Alert you: summary + draft reply
                                                                                                  ├─ support? ─▶ Draft emailed to you only
                                                                                                  └─ billing / spam-or-other ─▶ Logged only, nothing sent
```

**Who it's for:** any busy business owner drowning in email — trades, clinics, agencies, shops, consultants — who can't afford to miss a lead but has no assistant to triage the inbox.

**Setup time:** ~25 minutes · **Difficulty:** Easy–Medium

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account (Gmail) | The watched inbox; also sends the alerts and drafts | Free |
| A Google Sheet | The `Inbox Log` — your searchable record of every classified email | Free |
| An OpenAI API key | Powers the classification and draft replies (`gpt-4o-mini`) | Pay-as-you-go; a short email classification costs a fraction of a cent. A small top-up (e.g. $5) lasts a long time at small-business volumes. You can swap the HTTP node for another AI provider if you prefer |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **one tab**, named exactly:
   - `Inbox Log` with headers in row 1: `Timestamp | From | Subject | Category | Urgency | Summary | Draft Reply`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into the *Log to Inbox Sheet* node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Gmail account.** Click the *New Email Arrives* trigger → credential dropdown → **Create new** → sign in with the Google account whose inbox you want watched. Repeat on the Gmail send nodes (*Alert Owner — New Lead*, *Send Draft to Owner — Support*, *Alert — AI Step Failed*) — you can reuse the same account; n8n stores the trigger and send credentials separately. Do the same for the Google Sheets node.
4. **Add the OpenAI credential.** Create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer YOUR_OPENAI_API_KEY` — and select it in the *AI — Classify & Draft* node.
5. **Edit the CONFIG node.** Set:
   - `config_business_name` → your business name (used in the AI's drafting voice)
   - `config_owner_email` → your email (support drafts and failure alerts go here)
   - `config_alert_email` → where **lead alerts** go (can be the same as the owner email, or a phone-checked inbox)
   - `config_services` → 1–2 sentences on what you sell, and what counts as a lead for you. This is the single biggest lever on classification quality — be specific.
6. **Check the trigger's field mapping (one minute, worth it).** Open *Normalise Email* and look at the expressions. The Gmail Trigger's output shape varies slightly with the connected account and the `simple` setting, so the expressions use defensive fallbacks (`from.value[0].address`, then `from_email`, then `sender`, and similar). After your first test run (step 8), open the *Normalise Email* node's output: if any field came through empty, click into that assignment and remap it from the trigger's actual output fields using the expression editor's field picker.
7. **Filter the noise (recommended).** If the connected inbox also receives newsletters, order notifications or personal mail, open the *New Email Arrives* trigger and use its filter / label options (for example, only process mail carrying a specific Gmail label) so only mail you care about is classified. Cleanest option of all: connect a mailbox dedicated to business enquiries.
8. **Test it.** With the workflow open, click **Execute workflow**, then — from a **different email account** — send the connected inbox an obvious enquiry, e.g. subject `Price for a new website?` with a couple of sentences asking about cost and timing. Check: a row appears in `Inbox Log` with category `lead`, and a lead alert (with summary and draft reply) lands in the alert inbox. Send a second test written like an existing customer with a problem, and confirm the draft arrives addressed to **you**, not the sender.
9. **Go live.** Flip the workflow to **Active** (top-right toggle). From now on the trigger polls the inbox every minute on its own.

## Credentials used

- **Gmail OAuth2** — watch the inbox (trigger) and send the lead alert, support draft and failure alert (×3 send nodes)
- **Google Sheets OAuth2** — append one row per email to the `Inbox Log` (×1 node)
- **OpenAI API key (Header Auth)** — classification and draft replies (×1 HTTP node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only read or send from an account *you* connect.

## Good to know

- **The Gmail Trigger polls — it is not a true push.** At the default every-minute polling, a lead alert can lag by up to the poll interval plus a few seconds of AI time. Fast, but don't promise customers an instant human reply on the strength of it.
- **The AI will sometimes misclassify.** A billing question can read like support; a vague one-liner can land in spam-or-other. That is precisely why this free version **never auto-sends anything to a customer** — lead alerts and support drafts go to the owner only, and every email (right or wrong) is in the `Inbox Log` sheet where a 2-minute scan catches mistakes.
- **Only the newest message is classified, on an excerpt.** The workflow reads the first ~1,500 characters of the email text. Long threads, quoted history and attachments are not read — open the original email before replying to anything complicated.
- **Billing is logged, not routed.** Billing emails land in the `Inbox Log` like everything else; this free version doesn't forward them to an accounts address or label them in Gmail. If you want that routing, see the paid build below.
- **The draft reply is a starting point, not a send button.** The AI is told never to promise prices, dates or timelines that weren't given — but read every draft before you copy-paste it. Your name goes on the reply, not the workflow's.
- **Spam-or-other is deliberately silent.** No alert, no draft — just the log row. If the log shows real enquiries landing there, tighten your `config_services` wording first; it fixes most misclassifications.

## Want the grown-up version?

The RozContent paid build adds: billing routing to your accounts inbox, automatic Gmail labels per category, WhatsApp lead alerts for high-urgency mail, CRM sync so a lead becomes a deal (not just a log row), draft replies written from your past sent emails so they sound like you, and monitoring so a broken step gets fixed before it costs you a client.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
