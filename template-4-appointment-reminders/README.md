# Template 4 — Appointment Reminder & No-Show Reducer

**The problem:** every no-show is an hour you could have sold to someone else. Clients rarely ghost you on purpose — they forget, because the booking was made days ago and nothing reminded them. And reminding everyone by hand is exactly the kind of chore that gets skipped on a busy day.

**What this workflow does:** every hour it reads your appointments sheet and sends the two reminders that actually move no-show rates — one **24 hours before** and one **2 hours before** — marking each one in the sheet so nobody is reminded twice. Each morning it also sends **you** a single email listing tomorrow's bookings.

```
Every hour ─▶ Read Appointments ─▶ Plan ─┬─ 24h away? ─▶ Reminder email ─▶ Mark sheet
                                          ├─ 2h away? ──▶ Nudge email ────▶ Mark sheet
                                          └─ Morning? ──▶ Tomorrow's list ─▶ To you
```

**Who it's for:** clinics, dental and aesthetic practices, physios, salons, barbers, consultants — anyone whose diary is their income.

**Setup time:** ~25 minutes · **Difficulty:** Easy–Medium (one sheet, kept tidy)

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet is your appointment book; Gmail sends the reminders | Free |
| A habit of putting every booking in the sheet | The workflow can only remind about appointments it can see | 30 seconds per booking |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes left to right.
2. **Create the Appointments sheet.** One tab, named exactly `Appointments`, with these headers in row 1:
   `Client Name | Email | Service | Appointment Date | Appointment Time | Status | Reminder 24h Sent | Reminder 2h Sent`
   
   ⚠️ **Formats matter:** dates as `YYYY-MM-DD` (e.g. `2026-10-20`) and times as 24-hour `HH:MM` (e.g. `14:30`). The workflow reads them literally — `20/10/2026` or `2:30 PM` will be skipped.
   
   Copy the sheet's ID from its URL and paste it into the *Read Appointments* node and both *Mark as Sent* nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the Google Sheets nodes (×3) and Gmail nodes (×3) and connect your account on each. You can reuse the same Google account; n8n treats Sheets and Gmail credentials separately.
4. **Edit the CONFIG node.**
   - `config_owner_email` → the inbox that gets the morning list
   - `config_business_name` → your business name (used in the reminder wording)
   - `config_digest_hour` → the hour your morning list is sent, 24-hour clock (default `8`)
5. **Set the workflow timezone.** Top-right ⋯ menu → **Settings** → set the timezone to your business timezone. All the "24 hours before" maths uses it.
6. **Read the two reminders and make them yours.** Open *Send 24h Reminder* and *Send 2h Reminder* and adjust the tone. Note the ask in both: *reply to this email if you can't make it* — a cancellation you know about beats a no-show every time.
7. **Test it.** Add three fake rows for a client email you control: one appointment starting ~24 hours from now, one ~2 hours from now, one tomorrow at any time. Click **Run Manually**. Check: the 24h and 2h reminder emails arrived, and the sheet now shows timestamps in the `Reminder 24h Sent` / `Reminder 2h Sent` columns. (The morning list only fires on the run that lands inside your digest hour — to see it during testing, temporarily set `config_digest_hour` to the current hour and run again.)
8. **Go live.** Flip the workflow to **Active**. From now on: every booking goes in the sheet, every cancellation gets Status `Cancelled`.

## Credentials used

- **Google Sheets OAuth2** — read the appointments, stamp reminders as sent (×3 nodes)
- **Gmail OAuth2** — the two client reminders and your morning list (×3 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **Nothing is held in memory.** There are no long-running Wait executions — the "already reminded" flags live in the sheet itself, so restarting n8n never causes a duplicate reminder.
- **The windows are deliberate:** 23–25 hours and 1–3 hours before the appointment, checked hourly. A reminder can land up to an hour early or late. That is plenty to cut no-shows — it is not a precision alarm clock.
- **A missed hour is a missed reminder.** If n8n is down when a 2h reminder falls due, it won't be sent late — it will be skipped. The morning list of tomorrow's bookings is your safety net.
- Rows with Status `Cancelled` are never reminded. Any other status (Booked, Confirmed, blank) counts as a live booking.
- Book two appointments for the same client at the same date and time and each row is handled separately — one reminder per row.

## Honest limits

Email is the quietest channel there is. Plenty of clients will not open an email two hours before their appointment — the paid version below exists precisely because WhatsApp/SMS get read. This free template will still catch every client who *does* read email, and gives you the morning list, which alone is worth having.

## Want the version clients actually see in time?

The RozContent paid build adds: WhatsApp and/or SMS reminders (where last-minute messages get read), a confirm-or-cancel link inside every reminder so clients answer with one tap, an automatic rebooking nudge when someone cancels, and monitoring so a broken step is fixed before it costs you a booking.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
