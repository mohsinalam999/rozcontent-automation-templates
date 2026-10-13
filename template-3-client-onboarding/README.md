# Template 3 — Client Onboarding (Form → Folder → Welcome)

**The problem:** a client says yes… and then onboarding happens in a scatter of emails. Folders get made late (or twice), the client doesn't know what to send you, you forget a step, and their first impression of your "organised studio" is chaos.

**What this workflow does:** the client fills in **one kickoff form** (hosted by n8n itself — no extra form tool needed). Instantly and automatically:

1. Their **Drive folder structure** is created: a client folder with `01 — Brief & Assets`, `02 — Work in Progress`, `03 — Deliverables & Invoices`
2. A **welcome email** goes out with the folder link and clear next steps
3. A seven-task **onboarding checklist** lands in your tracker sheet
4. **You** get a notification with everything in one place

If folder creation fails, you get a specific alert instead of silence — the client never falls through the crack quietly.

```
Client submits kickoff form ─▶ Clean up ─▶ Create client folder ─▶ (failed? → alert you)
                                                  ├─▶ Create 3 subfolders
                                                  ├─▶ Welcome email to client
                                                  ├─▶ Notify you
                                                  └─▶ Build checklist ─▶ Rows in Onboarding Tracker
```

**Who it's for:** freelancers, studios and agencies that onboard clients manually — designers, marketers, consultants, content and automation services.

**Setup time:** ~20 minutes · **Difficulty:** Easy

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow *and hosts the form* | Free tier is fine |
| A Google account | Drive folders, Sheets tracker, Gmail emails | Free |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes.
2. **Create the parent folder.** In Google Drive, make one folder, e.g. `Clients`. Open it and copy its folder ID from the URL (the string after `/folders/`). Paste it into the *Create Client Folder* node, replacing `PASTE_YOUR_CLIENTS_PARENT_FOLDER_ID`.
3. **Create the tracker sheet.** New Google Sheet → tab named exactly `Onboarding Tracker` → headers in row 1: `Created | Client | Contact Name | Contact Email | Package | Task | Status`. Paste the sheet ID into the Sheets node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
4. **Connect your Google account** on the Drive (×4), Sheets (×1) and Gmail (×3) nodes. The file ships with placeholder credentials; n8n will prompt you to connect your own.
5. **Edit the CONFIG node.** Your business name (used in the welcome email) and the owner email for notifications.
6. **Make the form yours.** Open the *New Client Kickoff Form* node: change the package options to your real services, add fields you actually need (website URL, start date, goals). If you rename a field, update the matching line in *Normalise Client*.
7. **Make the checklist yours.** The seven tasks live in the *Build Onboarding Checklist* code node, in plain English — edit them to match your real onboarding.
8. **Test end-to-end.** Activate the workflow, open the form's Production URL, and submit a fake client using **your own** email. Verify: folder + 3 subfolders created, welcome email received, 7 checklist rows in the tracker, owner notification received.
9. **Use it.** Send the form link to every new client the moment they say yes — put it in your proposal and your "welcome" message.

## Two honest manual touches

By design, two things stay human in this free version:

- **Sharing the folder.** The welcome email links the folder but doesn't grant access — you share it with the client's email yourself, so you always control who sees what.
- **The kickoff call.** The automation tells the client it's coming; booking it is yours.

## Credentials used

- **Google Drive OAuth2** — create the client folder + subfolders (×4 nodes)
- **Google Sheets OAuth2** — append the checklist rows (×1 node)
- **Gmail OAuth2** — welcome email, owner notification, failure alert (×3 nodes)

## Good to know

- The form is public to anyone with the link. That's normal for a kickoff form (no sensitive questions on it) — and the workflow never asks for passwords; the welcome email explicitly tells clients to share access by invitation instead.
- Repeat submissions from the same business create a second folder. A quick glance at Drive before big projects keeps things tidy.

## Want onboarding that runs itself end to end?

The RozContent paid build adds: automatic folder sharing with the right people, a kickoff-call booking link wired in, invoice creation from your template, package-specific checklists, a client-facing progress view, and monitoring under a care plan.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
