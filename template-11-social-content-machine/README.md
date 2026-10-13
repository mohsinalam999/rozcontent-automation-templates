# Template 11 — AI Social Media Content Machine

**The problem:** you know your business should post every week. You also know how most weeks actually go — Monday's blank page wins, the accounts stay quiet, and the one post that does go out was written in a panic. The ideas were never the problem; the weekly writing chore was.

**What this workflow does:** every Monday at 08:00 it reads a simple content calendar that *you* fill in. For each topic you marked `Planned`, it writes four platform-shaped drafts — LinkedIn, X, Facebook and Instagram, each in that platform's register and your brand voice — and files them in a Draft Queue sheet. Then it emails you that the week's drafts are waiting for review.

**How this differs from Template 2:** Template 2 *reacts* — it watches your blog's RSS feed and drafts posts when a new article appears. Template 11 *plans* — it works from a calendar you control, so it covers the posts that are not tied to articles: tips, offers, behind-the-scenes, seasonal content, questions for your audience. Plenty of owners run both.

**The design rule, in bold: nothing ever auto-posts.** AI drafts; a human reads, tweaks and posts. Your social accounts never hand posting rights to an automation, and a bad draft dies in a spreadsheet instead of on your timeline.

```
Monday 08:00 ─▶ CONFIG ─▶ Read Content Calendar ─▶ Status = Planned?
                                                            ├─ Yes ─▶ AI writes 4 posts per topic
                                                            │              ├─▶ Split into rows (1 per platform)
                                                            │              ├─▶ Save to Draft Queue sheet
                                                            │              ├─▶ Email you: drafts waiting
                                                            │              └─ (AI failed? → alert email instead)
                                                            └─ No ───▶ workflow quietly stops (no drafts, no email)
```

**Who it's for:** agencies, creators, and businesses that post inconsistently — anyone whose socials go quiet because writing the week's posts is a chore nobody owns.

**Setup time:** ~25 minutes · **Difficulty:** Easy–Medium (one API key)

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The content calendar + draft queue (Sheets) and the notification emails (Gmail) | Free |
| An OpenAI API key | Writes the drafts | Pay-as-you-go — typically well under a cent per topic on `gpt-4o-mini`, so a small credit top-up lasts a long time at weekly volumes. There is no meaningful free API tier; budget a few dollars to start |
| 15 minutes a week from a human | Filling in the calendar's topics and angles — the machine writes, it does not decide strategy | Your time |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Content Calendar` with headers in row 1: `Week Of | Topic | Angle | Status`
   - `Draft Queue` with headers in row 1: `Timestamp | Topic | Platform | Draft | Status`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes (*Read Content Calendar* and *Save Drafts to Review Queue*), replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the other Sheets node and the Gmail nodes (you can reuse the same Google account; n8n treats Sheets and Gmail credentials separately).
4. **Add the AI key.** Create an n8n **Header Auth** credential: Name `Authorization`, Value `Bearer YOUR_OPENAI_API_KEY`. Select it in the *AI — Write the Week's Posts* node.
5. **Edit the CONFIG node.** Set:
   - `config_business_name` → your business name (used in the AI's instructions)
   - `config_brand_voice` → one or two sentences that keep the drafts sounding like you, e.g. "Plain English, specific over clever, no hype" — this line does more for draft quality than anything else in the workflow
   - `config_owner_email` → the inbox that should be told when drafts are waiting (and alerted if the AI step fails)
   - `config_current_offer` → this month's promotion, to be woven in lightly where it fits — or leave it as `None`, in which case the AI is told to promote nothing and never to invent an offer
6. **Fill in the calendar.** Add a row per topic in `Content Calendar`: the week, the topic, the angle you want taken, and `Status` set to `Planned`. Rows with any other status are ignored.
7. **Test it.** Set one calendar row to `Planned`, then execute the workflow manually (open the *Weekly — Monday 08:00* node → **Execute step**, or run the whole workflow once). Check: four rows in `Draft Queue` — linkedin, x, facebook, instagram — each marked `Pending review`, plus a notification email telling you how many topics were processed and how many drafts were filed.
8. **Go live.** Flip the workflow to **Active** (top-right toggle). It now runs every Monday at 08:00 (change the day/time in the schedule node if your week starts differently).

## The weekly habit

Open the `Draft Queue`, read a draft, edit it in the cell if needed, post it to the platform yourself, then mark it done. When you have reviewed a topic's drafts, flip its `Content Calendar` row from `Planned` to `Drafted` — that one small step is what stops next Monday's run from drafting the same topic again.

## Credentials used

- **Google Sheets OAuth2** — read the Content Calendar and append drafts to the Draft Queue (×2 nodes)
- **Gmail OAuth2** — send the "drafts waiting" notice and the AI-failure alert (×2 nodes)
- **HTTP Header Auth** — your OpenAI API key (×1 node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only act through accounts *you* connect.

## Good to know

- **Drafts only — always.** This workflow never posts to LinkedIn, X, Facebook or Instagram. Platform auto-posting needs each platform's own developer app and approvals; that setup is part of the paid build, not a setting in this template.
- **A human must fill the Content Calendar.** The machine writes the posts; it does not decide your strategy, topics or angles. Garbage calendar in, garbage drafts out.
- **AI drafts need a human edit pass.** The AI is instructed never to invent facts, statistics or offers beyond your `config_current_offer` — but voice, facts and local references still need your eyes before anything is posted. That review is the feature, not overhead.
- **Posting happens by hand from the Draft Queue.** Two minutes per post: read, tweak, post, done.
- **No `Planned` rows = a quiet stop.** If nothing is marked `Planned`, the workflow ends without calling the AI or emailing you. No noise, no cost.
- **Rows left as `Planned` get drafted again next Monday.** Flip reviewed rows to `Drafted` (step 8's habit) — this free version does not update the calendar for you.
- One AI call runs per planned topic, so a big calendar costs a little more and takes a little longer. At small-business volumes — a handful of topics a week — both are negligible.

## Want the grown-up version?

The RozContent paid build adds: auto-posting to your platforms (each platform's developer app set up and approved for you), calendar rows flipped to `Drafted` automatically after drafting, topic and angle suggestions so the calendar fills itself in for your approval, image suggestions per post, multiple brands/businesses in one workflow, a review board instead of a sheet, and monitoring so a broken step gets fixed before it costs you a week of posts.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
