# Template 14 — Meeting Notes → Tasks

**The problem:** every meeting ends with good intentions scattered through somebody's notebook. The summary never gets written, the tasks live in three different heads, and two weeks later nobody is sure who agreed to do what — or by when.

**What this workflow does:** you send it the raw notes (or a transcript) from a meeting, and it does the write-up for you. AI extracts a short summary, the decisions made, and every task with its owner and due date. The tasks land in a Google Sheet task list, the summary is emailed to you, and the meeting is recorded in a meeting log — minutes after the call ends, not days later.

```
Form / shortcut / curl ─▶ Webhook ─▶ Clean up ─▶ CONFIG ─▶ Notes included?
                                                                 ├─ Yes ─▶ AI extract ─▶ Parse ─┬─▶ Task rows ─▶ Save to Task List
                                                                 │              │                  └─▶ Format ─┬─▶ Email summary to owner
                                                                 │              │                             └─▶ Log meeting
                                                                 │              └─ error ─▶ Alert you
                                                                 └─ No ───▶ Alert you, AI not called
```

**Who it's for:** agencies, consultants, and small teams with weekly calls — anyone whose meetings produce actions that currently live in notebooks and memory.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet holds the Task List and Meeting Log; Gmail sends the summary and alerts | Free |
| An OpenAI API key | The AI extraction step (summary, decisions, tasks) | Pay-as-you-go; a mini-class model costs a fraction of a cent per normal meeting, but you do need a funded OpenAI API account — there is no free API tier for this |
| Something that can POST JSON to a URL | Feeds the notes in — a form (Tally/Typeform) with a webhook, a note-app shortcut, or curl | Free options exist |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Task List` with headers in row 1: `Timestamp | Meeting | Meeting Date | Task | Owner | Due | Status`
   - `Meeting Log` with headers: `Timestamp | Meeting | Meeting Date | Attendees | Summary | Task Count`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail nodes (you can reuse the same Google account; n8n treats Sheets and Gmail credentials separately).
4. **Add your OpenAI key.** Create a **Header Auth** credential — Name: `Authorization`, Value: `Bearer YOUR-OPENAI-API-KEY` — then open the *AI — Extract Summary & Tasks* node and select it.
5. **Edit the CONFIG node.** Set:
   - `config_business_name` → your business name (used in the AI prompt)
   - `config_owner_email` → the inbox that should receive meeting summaries and alerts
   - `config_team_note` → the names and roles of your regular attendees, e.g. `Sara — project lead; Ahmed — developer; Priya — designer`. This genuinely helps: it is how the AI matches a name mentioned in the notes to the right task owner.
6. **Point your notes source at the webhook.** Open the *Meeting Notes Webhook* node → copy the **Production URL** → set your form, shortcut or script to POST these JSON fields to it: `title`, `date`, `attendees` (comma-separated text), `notes` (the raw notes or transcript text). (No source wired yet? Test first with step 7, then connect it.)
7. **Test it.** With the workflow open, click **Execute workflow**, then send this sample meeting — e.g. with curl:
   ```bash
   curl -X POST https://YOUR-N8N-URL/webhook-test/meeting-notes \
     -H "Content-Type: application/json" \
     -d '{"title":"Weekly client sync — Acme website","date":"2026-10-13","attendees":"Sara, Ahmed, Priya","notes":"Priya opened by confirming the new homepage design is approved and goes live on Friday.\nSara said the client has signed off the logo, so there will be no more design changes this month.\nWe agreed to move the launch email campaign from Tuesday to Thursday to avoid the public holiday.\nAhmed will rebuild the checkout page mock-up and share it by Wednesday.\nSara will send the final price list to the client by Monday morning.\nPriya will book the photographer for the product shoot next week, date to be confirmed.\nWe decided to pause the blog redesign until the checkout work is finished.\nAhmed raised that the test account still has no payment method set up — it needs fixing before launch, no date set."}'
   ```
   Check: task rows appear in `Task List` (Ahmed's checkout mock-up, Sara's price list, Priya's photographer booking — and the payment-method fix, which was raised without a named owner, so don't be surprised if it lands as `Unassigned`), one row in `Meeting Log`, and a summary email in the owner inbox listing the decisions (homepage goes live Friday, campaign moved to Thursday, blog redesign paused).
8. **Go live.** Flip the workflow to **Active** (top-right toggle), then swap your notes source to the **Production URL** (`/webhook/meeting-notes`, without `-test`).

## Credentials used

- **Google Sheets OAuth2** — append task rows and meeting log rows (×2 nodes)
- **Gmail OAuth2** — send the summary email, the empty-submission alert and the AI-failure alert (×3 nodes)
- **OpenAI API key (Header Auth)** — the extraction call (×1 node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **This workflow does not record or transcribe audio.** Feed it notes or a transcript you already have — Zoom/Meet/Otter exports work well. (The paid build wires a transcription step in.)
- **Extraction quality = notes quality.** Cryptic shorthand produces vague tasks; a few full sentences per topic — like the sample above — produce clean ones.
- **Owners and due dates are the AI's reading of your notes.** The AI is instructed never to invent a task or decision, and to mark ownerless tasks `Unassigned` rather than guess — but still verify the Task List before treating its rows as commitments.
- **Very long transcripts may exceed the model's context.** Split monster meetings into parts and send each part separately.
- **A submission with empty notes costs you nothing:** the workflow alerts you and stops before the AI step, so a misfired form never burns an API call.
- **The free version emails the owner only**, who forwards the summary to attendees. If zero tasks are captured, no rows are appended to the Task List — but the summary email and the Meeting Log entry still happen.

## Want the grown-up version?

The RozContent paid build adds: automatic transcription wired in (Zoom / Meet / Otter exports, or a Whisper step for raw audio), a personal email to each attendee with only their own tasks, due-date reminders and an overdue-task chase, calendar invites for follow-ups, and updating a task's status in the sheet when it's replied to or completed — turning the Task List into a living tracker instead of an archive.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
