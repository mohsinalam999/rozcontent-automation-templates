# Template 2 — Content Repurposer (Blog → Social Drafts)

**The problem:** your business writes a good blog post, publishes it… and never tells anyone. The post sits there while your LinkedIn, X and Facebook accounts go quiet for another week — because turning one article into three platform-appropriate posts is exactly the kind of small job nobody has time for.

**What this workflow does:** every morning it checks your blog's RSS feed. For each new post it drafts three promotional versions — LinkedIn, X and Facebook, each written in that platform's register — and files them in a Google Sheet review queue. Then it emails you that drafts are waiting.

**The design rule, in bold: nothing ever auto-posts.** AI drafts; a human reads, tweaks and posts. Your social accounts never hand posting rights to an automation, and a bad draft dies in a spreadsheet instead of on your timeline.

```
Daily 9am (or manual run) ─▶ Fetch RSS ─▶ Clean up ─▶ Published in last 24h?
                                                            └─ Yes ─▶ AI drafts 3 variants
                                                                          ├─▶ Split into rows
                                                                          ├─▶ Save to Review Queue sheet
                                                                          ├─▶ Email you: drafts waiting
                                                                          └─ (AI failed? → alert email instead)
```

**Who it's for:** businesses and creators who publish articles but don't promote them; agencies producing content for clients; anyone running a blog + socials with a small team.

**Setup time:** ~25 minutes · **Difficulty:** Easy–Medium (one API key)

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A blog with an RSS feed | The trigger source (WordPress: `yoursite.com/feed`; most platforms have one) | — |
| An OpenAI API key | Writes the drafts | A fraction of a cent per post on a mini-class model |
| A Google account | The review queue (Sheets) + the notification (Gmail) | Free |

*No OpenAI key? The drafting node is a standard HTTP call — the workflow README in n8n (sticky notes) explains how to swap in another AI provider as long as it returns JSON with `linkedin` / `x` / `facebook` keys.*

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes.
2. **Set your feed.** Open *Fetch Blog RSS* and replace `https://your-blog.com/feed/` with your real feed URL.
3. **Create the review sheet.** New Google Sheet → tab named exactly `Review Queue` → headers in row 1: `Created | Platform | Draft | Source Title | Source URL | Status`. Copy the sheet ID from its URL and paste it into the Sheets node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
4. **Connect Google.** Click the Sheets and Gmail nodes and connect **your** Google account where prompted (placeholder credentials ship in the file; n8n will ask you to replace them).
5. **Add the AI key.** Create an n8n **Header Auth** credential: Name `Authorization`, Value `Bearer YOUR_OPENAI_API_KEY`. Select it in the *AI Draft — All 3 Variants* node.
6. **Edit the CONFIG node.** Set your business name, your **brand voice** (one or two sentences — this is what keeps the drafts sounding like you, e.g. "Plain English, specific over clever, no hype"), and the email that should be told when drafts are waiting.
7. **Test.** Publish a post (or use any post from the last 24 hours), then click **Execute workflow** on the *Run Manually* trigger. Check the `Review Queue` tab: you should see three rows — linkedin, x, facebook — each marked `Pending review`, plus a notification email.
8. **Go live.** Flip the workflow to **Active**. It now runs daily at 9am (change the time in the *Daily at 9am* node if you prefer another slot).

## How the review habit works

Open the sheet, read a draft, edit it in the cell if needed, post it to the platform yourself, then change its `Status` to `Posted`. That's the whole system — two minutes per post, and your blog finally promotes itself.

## Credentials used

- **HTTP Header Auth** — your OpenAI API key (one node)
- **Google Sheets OAuth2** — append draft rows (one node)
- **Gmail OAuth2** — the "drafts waiting" notice and the AI-failure alert (two nodes)

## Good to know

- **De-duplication is a simple 24-hour window.** If the workflow runs twice within a post's first day, you may get a duplicate draft set — delete the extra rows. (The paid build keeps a proper already-drafted store.)
- **The AI is instructed not to invent facts** beyond the excerpt — but read before posting anyway. That review is the feature, not overhead.
- Tighter drafts: lengthen the excerpt slice in *Normalise Post* (default 1,200 characters) or sharpen the brand-voice line in CONFIG.

## Want it tuned to your voice and wired into your stack?

The RozContent paid build adds: true de-duplication, per-platform scheduling suggestions, image/cover suggestions per post, a Notion or Airtable review board instead of a sheet, multiple blogs/brands in one workflow, and monitoring with a care plan.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
