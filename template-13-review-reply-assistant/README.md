# Template 13 — Google Review Reply Assistant

**The problem:** Google reviews pile up unanswered — the glowing 5-star ones get no thanks, and the angry 1-star ones sit there in public, unanswered, doing quiet damage. Replying well takes time nobody has, so most businesses reply late or not at all. This template is the replying half of the pair: **Template 5 (Review Request Engine) asks for the reviews; this one answers them.**

**What this workflow does:** twice a day it reads the reviews you pasted into a Google Sheet, asks AI to draft a reply to each one in your voice — warm and specific for good reviews, calm and never defensive for bad ones — saves the drafts to an approval tab, and emails you one short digest. You read, tweak, paste the reply into Google Business Profile, done. About two minutes per review, and nothing is ever posted without you.

```
Reviews tab ─▶ (twice daily) ─▶ Keep "Needs reply" rows ─▶ AI drafts each reply
                                                              ├─ (AI error? → alert you)
                                                              └─▶ Save to Reply Drafts tab ─▶ Digest email to you
                                                                                                    │
                                          You: read → edit → paste into Google Business Profile → mark Approved
```

**Who it's for:** restaurants, clinics, salons, and any local business whose next customer reads its Google reviews before deciding.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet holds reviews and drafts; Gmail sends the digest | Free |
| An OpenAI API key | Drafts the replies (gpt-4o-mini) | Pay-as-you-go — a fraction of a cent per reply |
| A Google Business Profile | Where review notification emails come from, and where final replies get pasted | Free |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Reviews` with headers in row 1: `Date | Reviewer | Stars | Review Text | Draft Reply | Status`
   - `Reply Drafts` with headers: `Timestamp | Reviewer | Stars | Review Text | Draft Reply | Status`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail nodes (n8n treats Sheets and Gmail credentials separately, even for the same account).
4. **Add your OpenAI key.** Create a credential of type **Header Auth** — Name: `Authorization`, Value: `Bearer YOUR_OPENAI_API_KEY` — then open the *AI — Draft the Reply* node and select it. (Prefer another AI provider? Swap that one node; the prompt asks for JSON with a single `draft_reply` key.)
5. **Edit the CONFIG node.** Set:
   - `config_business_name` → your business name (used in the AI prompt and the digest)
   - `config_owner_name` → the human who signs replies, e.g. `Sara, Manager`
   - `config_owner_email` → the inbox that should receive the digest and failure alerts
   - `config_tone_note` → how replies should sound, e.g. `warm, brief, never defensive`
   - `config_review_link_note` → your reminder of where replies get posted (the default text is fine for most)
6. **Test it.** Paste these two fake reviews into the `Reviews` tab as rows (leave *Draft Reply* empty):
   - `2026-10-13 | Amelia R. | 5 | The oat latte was perfect and the staff remembered my name from last week. Best café on the street. | | Needs reply`
   - `2026-10-13 | Danish K. | 2 | Waited 40 minutes for our food and it arrived lukewarm. Nobody checked on us. | | Needs reply`

   Then click **Run Manually → Execute**. Check: two new rows in `Reply Drafts` with Status `Awaiting approval`, and one digest email. Read both drafts — the 5-star one should be short, warm and mention the oat latte or being remembered; the 2-star one should apologise plainly, make no excuses, offer nothing, and invite Danish to make contact offline.
7. **Go live.** Flip the workflow to **Active** (top-right toggle). From now on, whenever Google Business Profile emails you about a new review, paste it into `Reviews` with Status `Needs reply` — drafts will be waiting after the next 09:00 or 17:00 run.

## Credentials used

- **Google Sheets OAuth2** — read the `Reviews` tab, append to `Reply Drafts` (×2 nodes)
- **Gmail OAuth2** — send the digest and the AI-failure alert (×2 nodes)
- **OpenAI Header Auth** — the AI drafting call (×1 node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only act on accounts *you* connect.

## Good to know

- **Nothing is posted to Google automatically — by design, not by omission.** Posting replies through an API requires Google Business Profile API access, which Google approval-gates. This free version drafts and files; you paste the final reply into Google Business Profile yourself (~2 minutes per review).
- **Reviews reach the sheet by hand.** You paste them in from your Google Business Profile email notifications. The paid build connects the GBP API so reviews arrive on their own.
- **AI drafts for angry reviews are a starting point, not a send button.** Read every draft before posting — especially 1-star replies. The prompt forbids arguing, excuses, and invented offers, but the final judgement is yours.
- **Don't offer refunds or compensation in a reply unless that really is your policy.** The prompt is written to never invent offers; if you edit a draft by hand, keep that line.
- **Drafts are appended to a separate `Reply Drafts` tab instead of being written back into the `Reviews` tab by row number.** Simpler and more robust: re-sorting the Reviews tab can never land a draft on the wrong review. When you've posted a reply, mark its `Reply Drafts` row `Approved` and (optionally) paste the final text into the `Reviews` tab's *Draft Reply* column for your records.
- If the AI step ever fails (expired key, no credit), you get an alert email and the review stays `Needs reply`, so it is picked up again on the next run. If a single draft comes back unparseable, its row is marked `Awaiting approval — write by hand` and the digest counts it.
- If no reviews say `Needs reply`, the run stops silently — no AI cost, no empty digest emails.

## Want the grown-up version?

The RozContent paid build connects the Google Business Profile API directly: new reviews land in your sheet the moment they're posted (no copy-pasting from email notifications), approved replies post back to Google with one click instead of a manual paste, 1-star reviews trigger an instant alert to your phone instead of waiting for the twice-daily digest, and a monthly report shows your rating trend and response time.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
