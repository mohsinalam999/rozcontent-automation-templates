# Template 5 — Review Request Engine

**The problem:** your happiest customers would happily review you — they just never get asked at the right moment. By the time you remember, days have passed, the warm feeling has faded, and your competitor with 400 reviews wins the search. Asking by hand, one customer at a time, is the first task dropped on a busy week.

**What this workflow does:** once a day it checks your jobs sheet. When a job has been marked **Complete** for at least a day, the customer gets a polite review-request email with your Google (and TripAdvisor) links. Five days later, if no follow-up has been sent, **one** gentle reminder follows — never more. Every send is logged and stamped on the job row, so nobody is ever asked twice.

```
Daily ─▶ Read Jobs ─▶ Plan ─┬─ Complete ≥1 day, never asked ─▶ Request email ─▶ Stamp row ─▶ Log
                             └─ Asked 5+ days ago ────────────▶ One follow-up ─▶ Stamp row ─▶ Log
```

**Who it's for:** restaurants, clinics, salons, trades, home services — any business that lives on its reviews.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet tracks jobs; Gmail sends the requests | Free |
| A Google review link | From Google Business Profile → **Ask for reviews** | Free |
| Optional: a TripAdvisor listing | The emails carry a second link if you have one | Free |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes.
2. **Create the sheet.** One Google Sheet with **two tabs**, named exactly:
   - `Jobs` with headers in row 1: `Customer Name | Email | Job | Completion Date | Status | Review Request Sent | Follow-Up Sent`
   - `Review Log` with headers: `Timestamp | Email | Stage | Detail`
   
   Copy the sheet's ID from its URL and paste it into all four Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account** on the Sheets nodes (×4) and Gmail nodes (×2).
4. **Edit the CONFIG node.**
   - `config_owner_email` and `config_business_name`
   - `config_google_review_link` → your "Ask for reviews" link from Google Business Profile
   - `config_tripadvisor_link` → your TripAdvisor page link — **or leave it empty and delete the TripAdvisor line from both email nodes** (next step)
5. **Make the emails yours.** Open *Send Review Request Email* and *Send Gentle Follow-Up*. If you removed the TripAdvisor link, delete its line so customers don't see a blank. Adjust the tone — the ask works best short, warm and specific about the job.
6. **Build the one habit.** From now on, finishing a job means one row in `Jobs`: customer, email, job name, Completion Date as `YYYY-MM-DD`, Status `Complete`. (Completed / Done / Finished also count.) Everything after that row is automatic.
7. **Test it.** Add a fake job row using **your own** email: Status `Complete`, Completion Date **two days ago**. Click **Run Manually**. Check: the request email arrived, the row now has a timestamp under `Review Request Sent`, and a line appeared in `Review Log`. To test the follow-up without waiting five days, type a date six days ago into `Review Request Sent` and run again.
8. **Go live.** Flip the workflow to **Active** (top-right toggle). It runs daily at 10:00 — change the *Daily at 10am* node if you prefer a different hour.

## Credentials used

- **Google Sheets OAuth2** — read jobs, stamp sends, append the log (×4 nodes)
- **Gmail OAuth2** — the review request and the follow-up (×2 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **How a job row is found again:** the stamp nodes match on `Email + Job + Completion Date`. Keep the Job name distinct per customer (e.g. `Kitchen refit — INV-1042`, not just `Job`) so two similar rows never get confused.
- **The timing is the point.** A day after completion, the experience is fresh but the customer has had time to enjoy the result. Asking in the first hour feels pushy; asking after a month gets shrugs.
- **Why only one follow-up:** because that is what polite looks like, and because review platforms watch for pressure patterns. Two asks total, then silence — by design.
- Duplicate job rows for the same customer + job + date are treated as one job; the stamp lands on the first matching row.

## Honest limits

The workflow **cannot see whether a review was actually left** — it only knows what it sent. A customer who reviews you on day 2 may still receive the day-5 follow-up; the wording accounts for that ("please ignore this if you've already reviewed us"). It also never incentivises, filters or scripts reviews — no discounts-for-stars, no "only happy customers" routing. Platforms ban those tactics and delist businesses for them; this engine stays on the right side of that line permanently.

## Want reviews that get answered, not just requested?

The RozContent paid build adds: a notification to you the moment a new Google review lands, a drafted reply in your voice for every review (including the awkward ones — those replies are read by future customers), requests timed per job type, and a monthly summary of your rating trend.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
