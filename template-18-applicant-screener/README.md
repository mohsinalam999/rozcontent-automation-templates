# Template 18 — Job Applicant Screener

**The problem:** you post one job and get 60 applications. Reading every cover note properly takes an evening you don't have — so good people sit unread for a week, and the pile gets skimmed unfairly, best-written first rather than best-fit first.

**What this workflow does:** applicants submit a built-in n8n application form. Every application is saved to a Google Sheet, then an AI scores it **strictly against hiring criteria you write yourself** and files the score, verdict and reasoning in a second tab. Only applicants who meet your shortlist threshold trigger an email — to **you**, never to the applicant — so you start with the strongest files and still have every application on record.

```
Application form ─▶ Clean up ─▶ CONFIG criteria ─▶ Save application
                                                          │
                                                          ▼
                                              AI screens against YOUR criteria
                                                   ├─ error? ─▶ alert you (application is safe in the sheet)
                                                   └─ ok ─▶ Parse score ─▶ Save screening result
                                                                                  │
                                                                Score ≥ threshold? ─┤
                                                                   ├─ Yes ─▶ ⭐ Email YOU the highlights
                                                                   └─ No ───▶ Filed in Screening only. No email to anyone.
```

**Who it's for:** small businesses and agencies that hire occasionally — a shop, clinic, studio or agency taking on one role at a time and drowning in applications for it.

**Setup time:** ~25 minutes · **Difficulty:** Easy–Medium

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow and hosts the application form | Free tier is fine |
| A Google account | The Sheet stores applications + screening results; Gmail notifies you | Free |
| An OpenAI API key | Powers the screening step | Pay-as-you-go; screening one application costs a fraction of a cent with gpt-4o-mini. No free tier guarantee — new accounts may need a small credit top-up |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Applications` with headers in row 1: `Timestamp | Name | Email | Role | Experience (yrs) | Portfolio | Cover Note | Status`
   - `Screening` with headers: `Timestamp | Name | Email | Role | Score | Verdict | Summary | Strengths | Gaps`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail nodes (you can reuse the same Google account; n8n treats Sheets and Gmail credentials separately).
4. **Add your OpenAI key.** Open the *AI — Screen the Application* node → credential dropdown for HTTP Header Auth → **Create new**: name `Authorization`, value `Bearer YOUR-OPENAI-API-KEY` (your real key from platform.openai.com). The placeholder in the file is a dummy.
5. **Edit the CONFIG node — this is the step that decides whether this tool helps or harms.** Set:
   - `config_business_name` → your business name
   - `config_owner_email` → the inbox that should receive shortlist alerts
   - `config_shortlist_threshold` → `7` by default; the *Shortlist Score?* node reads this value directly, so change it here, not in the IF node
   - `config_role_criteria` → what a strong applicant for the **current** role looks like. Write it honestly, factually, and job-related only — see the worked example below.
6. **Test it.** Click **Execute workflow**, open the *Application Form* node's **Test URL**, and submit the **strong** sample application below. Check: a row in `Applications`, a row in `Screening` with a score/verdict, and — if the score is 7+ — a ⭐ email in the owner inbox. Then submit the **weak** sample: it should be filed in `Screening` and trigger **no** email.
7. **Go live.** Flip the workflow to **Active** (top-right toggle), then share the form's **Production URL** wherever the job is posted (your site, a job board, social posts).

### Writing criteria: good vs bad

The AI can only be as fair as the criteria you give it.

✅ **Good criteria** (factual, job-related, checkable against the application):

> "Strong: 2+ years of customer support experience; has used a helpdesk tool (Zendesk, Intercom or similar); cover note describes a specific difficult-customer situation and what they did; writes clearly. Weak: no support or people-facing experience and nothing transferable described; cover note is generic with no specifics. Deal-breaker: cannot work weekends — this role is weekend-based."

❌ **Bad criteria** (vague, personal, or impossible to judge fairly from an application):

> "Young and energetic, native speaker, from a good background, confident personality, premium university."

If a criterion is about *who someone is* rather than *what they can do and have done*, it doesn't belong in CONFIG. If you wouldn't defend a criterion to the applicant's face, don't put it in CONFIG.

### Sample applications for testing

**Strong sample** (expect a high score + the ⭐ email):
- Name: `Test Strong` · Email: your own email · Role: the role in your criteria
- Years of Experience: `4`
- Portfolio or LinkedIn URL: any real-looking URL
- Cover Note: a specific note that matches your criteria — e.g. for the support example above: *"I have done customer support for 4 years at an online retailer, mostly on Zendesk. Last winter a courier lost a customer's wedding order; I re-shipped it express at our cost, called her myself twice with updates, and she left a 5-star review naming me. I am available weekends."*

**Weak sample** (expect a low score, filed only, no email):
- Name: `Test Weak` · Years of Experience: `0`
- Cover Note: *"I am a hard worker and a fast learner looking for any opportunity. Please consider my application."*

## Credentials used

- **Google Sheets OAuth2** — append applications and screening results (×2 nodes)
- **Gmail OAuth2** — send the shortlist alert and the AI-failure alert to the owner (×2 nodes) — **never used to email applicants**
- **OpenAI API key (HTTP Header Auth)** — the screening call (×1 node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only act through accounts *you* connect.

## ⚖️ Fairness — read this before you use it

- **The AI score is a first-pass sorting aid only.** It can be wrong, and it can reflect biases in how an application is *worded* — confidence, native language, formatting — rather than how good the person actually is. A polished writer is not automatically a better hire.
- **A human must make every interview and rejection decision.** Use the score to decide reading order, not outcomes. Read the real applications — especially the ones just below your threshold.
- **This workflow sends NO email to any applicant, ever.** No automatic rejections, no automatic anything. Every applicant contact stays human, from your own inbox, in your own words.
- **Score only what's in the application and your stated criteria.** The AI is instructed to judge only job-related evidence and never to infer or score age, gender, nationality, religion, appearance or anything not in the application — but instructions are not guarantees, which is why the human read above is not optional.
- **Comply with your local employment law** on hiring, automated decision-making and how long you may keep applicant data. Delete applications you no longer need.

## Good to know

- "Maybe" and "pass" verdicts are filed in the `Screening` tab only — that's deliberate. Review that tab weekly while the role is open; scores near the threshold deserve a human read (see Fairness above).
- The threshold genuinely lives in CONFIG: the *Shortlist Score?* node compares the score against `config_shortlist_threshold`, so raising it to `8` or lowering it to `6` is a one-line edit.
- If the AI step fails (expired key, no credit), you get an alert email and the application is still saved in `Applications` with Status `New` — screen it by hand. Nothing is lost, it just isn't scored.
- One role at a time: the criteria in CONFIG describe the role you're currently hiring for. When the role changes, update the criteria *before* sharing the form again — or ask about the paid version with per-role criteria.
- The form is public once the workflow is active: anyone with the link can apply. That's the point of a job ad, but don't post the link anywhere the job itself isn't posted.

## Want the grown-up version?

The RozContent paid build adds: a separate criteria set per role (chosen from a dropdown on the form), a second independent AI pass that re-scores borderline applications before you see them, automatic interview scheduling links for applicants *you* approve, a weekly hiring summary, and monitoring so a broken step gets fixed before an applicant is lost.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
