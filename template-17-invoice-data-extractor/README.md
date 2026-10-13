# Template 17 — Receipt & Invoice Data Extractor

**The problem:** supplier invoices arrive by email, sit in the inbox, and get typed into a spreadsheet by hand — late, twice, or not at all. By tax time the expense record is a reconstruction, not a log.

**What this workflow does:** every time an invoice email arrives, it finds the PDF attachment, extracts the text, has an AI read out the vendor, invoice date, invoice number, subtotal, tax, total and currency, and appends it all as one tidy row in a Google Sheet. Anything it can't handle — no PDF, an unreadable PDF, a failed AI step, missing fields — it emails you about instead of silently dropping it.

```
Invoice email ─▶ Gmail Trigger ─▶ CONFIG ─▶ Find the PDF Attachment ─▶ PDF found?
                                                                          ├─ No ───▶ Alert you (handle by hand)
                                                                          └─ Yes ─▶ Extract PDF Text ─▶ (error? → alert you: couldn't read it)
                                                                                          │
                                                                                          ▼
                                                                              AI — Structure the Invoice ─▶ (error? → alert you)
                                                                                          │
                                                                                          ▼
                                                                              Parse Invoice ─▶ Save to Expenses Sheet
                                                                                          │
                                                                                          ▼
                                                                              Extraction complete?
                                                                                          ├─ Yes ─▶ Done, quietly
                                                                                          └─ No ───▶ Alert you (row saved, verify it)
```

**Who it's for:** freelancers, small businesses and bookkeepers drowning in emailed invoices who want a running expense log without the manual typing.

**Setup time:** ~25 minutes · **Difficulty:** Medium — the flow itself is simple; the binary-attachment plumbing is the fiddly part, and it is already wired for you. Just don't insert extra nodes between the trigger and the extract step without reading the Setup sticky.

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | Gmail receives the invoices and alerts; the Sheet is your expense log | Free |
| An OpenAI API key | Reads the invoice fields out of the extracted text | Pay-as-you-go; a fraction of a cent per invoice on a mini-class model. Free tiers/credits exist for new accounts, with limits |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **one tab**, named exactly:
   - `Expenses` with headers in row 1: `Timestamp | Vendor | Invoice Date | Invoice Number | Subtotal | Tax | Total | Currency | From Email | Subject | Complete?`
   
   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into the *Save to Expenses Sheet* node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Label your invoices in Gmail.** In Gmail, create a label called `invoices` (**Settings → Labels → Create new label**), then a filter (**Settings → Filters and blocked addresses → Create a new filter**) that applies that label to supplier emails — by sender, or by subject words like "invoice" / "receipt". This keeps your suppliers organised and makes the next step possible.
   - **Optional — restrict the trigger:** by default the *Invoice Email Arrives* trigger watches the whole inbox, which is fine if the inbox mainly receives invoices. If it doesn't, open that node and restrict it to the `invoices` label so personal email never enters the workflow. The node ships plain on purpose; the restriction is a two-click change in the node's own options once your label exists.
4. **Connect your Google account.** Click the Gmail Trigger node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail alert nodes and the Google Sheets node (you can reuse the same Google account; n8n treats Gmail and Sheets credentials separately).
5. **Add the AI key.** Create an n8n **Header Auth** credential: Name `Authorization`, Value `Bearer YOUR-OPENAI-API-KEY`. Select it in the *AI — Structure the Invoice* node.
6. **Edit the CONFIG node.** Set:
   - `config_owner_email` → the inbox that should receive the alerts (no-PDF, unreadable-PDF, AI-failure and check-this-invoice)
   - `config_business_name` → your business name (used in the AI prompt)
7. **Test it.** Email yourself a real **digital** invoice PDF — one generated by accounting software or an online supplier, not a photo of a paper receipt — at the connected Gmail address, with the subject "Test invoice". Click **Execute workflow** in n8n (or wait for the trigger to poll), then check: a row in `Expenses` with the vendor, date and total filled in, and no alert in the owner inbox. Then send one plain email with no attachment and confirm the *No PDF Found* alert arrives.
8. **Go live.** Flip the workflow to **Active** (top-right toggle). It now polls Gmail every minute.

## Credentials used

- **Gmail OAuth2** — the invoice trigger, plus the four alert emails (×5 nodes)
- **Google Sheets OAuth2** — append the expense row (×1 node)
- **HTTP Header Auth** — your OpenAI API key (×1 node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only read and send from an account *you* connect.

## Good to know

- **Digital PDFs work best.** The extract step reads the PDF's text layer. Invoices generated by accounting software work well; scanned paper receipts and photographed receipts usually extract little or no text — those land on the *Couldn't Read PDF* alert for manual entry. True OCR is in the paid build.
- **AI-read numbers need a spot-check.** Totals and dates are read by an AI model, not by your accounting software. Glance at each row against the PDF before relying on the sheet for tax — the `Complete?` column and the check-this-invoice alert exist to tell you where to look hardest.
- **One PDF per email** in this free version. If a supplier attaches three invoices to one email, only the first PDF is processed and the rest are ignored — ask suppliers to send one invoice per email, or see the paid version.
- **The sheet is an expense log, not accounting software.** No categories, no reconciliation, no duplicate detection in this free version. It gets the numbers out of your inbox and into a table; your bookkeeping still happens on top of it.
- **Don't break the binary chain.** Attachments travel on the n8n item's binary data. The *Find the PDF Attachment* Code node is written to carry that binary through untouched. If you insert a Set node between the trigger and the extract step, set it to include the other input fields — a plain Set node can drop the attachment and every invoice will report "no PDF".
- The trigger polls every minute, so a busy inbox means a lot of small executions. Restricting the trigger to the `invoices` label (step 3) keeps that in check.

## Want the grown-up version?

The RozContent paid build adds: true OCR so scanned and photographed receipts work too, multiple PDFs per email, duplicate-invoice detection, expense categories, a monthly totals summary emailed to you, and monitoring so a broken step gets fixed before your records have a gap in them.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
