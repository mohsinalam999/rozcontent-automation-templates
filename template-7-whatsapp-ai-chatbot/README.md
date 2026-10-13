# Template 7 — WhatsApp AI Chatbot for a Business

**The problem:** customers message your WhatsApp number with the same questions — opening hours, prices, how to book — at all hours, and slow replies lose sales. Hiring someone to sit on chat all day isn't an option for most small businesses.

**What this workflow does:** when a customer messages your WhatsApp business number, the message is logged to a Google Sheet, an AI writes a reply using **only** the facts you paste into a knowledge-base box (hours, services, prices, policies — it is told never to invent any), the reply is sent back through your WhatsApp provider, and you get an email the moment a chat looks like a lead or needs a human. You review everything in the Chat Log.

```
Customer WhatsApp ─▶ Provider webhook ─▶ Clean up ─▶ CONFIG ─▶ Log message to Sheet
                                                                      │
                                                                      ▼
                                              AI composes reply (knowledge base only)
                                                   ├─ error ─▶ Alert you by email
                                                   ▼
                                              Parse AI reply safely
                                                   ▼
                                        Needs human or a lead?
                                           ├─ Yes ─▶ Email you ──┐
                                           └─ No ────────────────┤
                                                                 ▼
                                                    Send WhatsApp reply ─▶ Log outcome to Sheet
```

**Who it's for:** clinics, restaurants, online stores — any business whose customers ask repeat questions on WhatsApp and expect a fast answer.

**Setup time:** ~35 minutes · **Difficulty:** Medium (the provider setup is the slow part, not the n8n part)

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| WhatsApp Business Platform access — Meta Cloud API directly, or a BSP such as Twilio / 360dialog / WATI | This is the only official way to automate WhatsApp; it delivers inbound messages to the webhook and sends your replies | Meta Cloud API has a free tier with limits; BSPs charge per conversation — check your provider's current pricing |
| A verified business phone number on that platform | The number customers message | Provider-dependent |
| A Google account | The Sheet is your chat log; Gmail sends owner alerts | Free |
| An OpenAI API key | Powers the AI reply step (`gpt-4o-mini`) | Paid, usage-based — a short chat reply costs a fraction of a cent; new accounts may need a small credit top-up |
| Your knowledge base as plain text | Hours, services, prices, policies — the only facts the bot may use | Free — you write it |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right.
2. **Get WhatsApp Business Platform access.** Either register on Meta's WhatsApp Business Platform (Cloud API) directly, or sign up with a BSP such as Twilio, 360dialog or WATI and connect your number there. Complete business verification and get your business phone number verified — nothing flows until this is done, and it is the slowest step.
3. **Create the Google Sheet.** Make a new sheet (any name) with **one tab**, named exactly:
   - `Chat Log` with headers in row 1: `Timestamp | From | Message | AI Reply | Outcome`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into both Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.

   *How logging works:* this template logs **two rows per chat** — *Log Message to Sheet* appends the inbound message **before** the AI step (so a message is never lost if the AI fails, with Outcome `Received — awaiting AI reply`), and *Log Outcome* appends a second row at the end with the reply that was sent and its outcome. Simpler and safer than trying to update the first row.
4. **Connect your accounts.** Click each Google Sheets / Gmail node → credential dropdown → **Create new** → sign in with your Google account. On *AI — Compose Reply*, create a Header Auth credential with your OpenAI API key (`Authorization` header, `Bearer YOUR_KEY`). On *Send WhatsApp Reply*, create a Header Auth credential with your WhatsApp provider's API key, and replace the placeholder URL `https://YOUR-WHATSAPP-PROVIDER.example/v1/messages` with your provider's real send-message endpoint.
5. **Edit the CONFIG node.** Set:
   - `config_business_name` → your business name
   - `config_owner_email` → the inbox that should get lead / human-handoff and error alerts
   - `config_knowledge_base` → paste your real facts: opening hours, services, prices, policies. Be complete and exact — the bot may **only** use this text, and it is instructed to flag a human rather than guess
   - `config_fallback_message` → the safe reply sent if the AI output can't be parsed
   - `config_whatsapp_number` → your verified business number
6. **Adjust the Normalise Message expressions to your provider.** Field paths differ per provider — be honest with yourself here, this step is not optional. Meta Cloud API nests the message deep inside the payload (under `entry[0].changes[0].value.messages[0]`, with the text at `…value.messages[0].text.body` and the sender at `…value.messages[0].from`); BSPs flatten it differently again. The *Normalise Message* Set node ships with defensive `||` fallbacks on the flatter fields (`from`, `text`, `message`, and similar). Send yourself one test message, open the webhook execution in n8n, look at the real payload your provider sent, and edit the four expressions (`sender_phone`, `sender_name`, `message_text`, `message_id`) in n8n's expression editor so each one pulls the right field from *your* payload.
7. **Adapt the send body if needed.** *Send WhatsApp Reply* ships with the Meta Cloud API message shape — `{to, type: "text", text: {body}}`. If your BSP expects a different shape or endpoint, change the URL and JSON body to match its docs; the `to` and reply-text values are the two things that must survive the change.
8. **Point the provider webhook at this workflow.** Open the *WhatsApp Inbound Webhook* node → copy the **Production URL** → paste it into your provider's inbound-message webhook settings (in Meta / your BSP dashboard), with whatever verification token your provider flow asks for.
9. **Test it.** Activate the workflow, then **send your own number a WhatsApp message** from your personal phone — e.g. "What are your opening hours?" Check: two rows appear in `Chat Log` (the inbound row and the outcome row), a reply arrives on your phone that matches your knowledge base word-for-fact, and no owner alert fired. Then test the human path: ask something that is deliberately *not* in your knowledge base — you should get the fallback-style reply on WhatsApp **and** a "chat needs you" email.
10. **Go live.** Leave the workflow **Active**, tell customers the number is open, and review the `Chat Log` daily for the first week or two — fix wrong or weak answers by improving the knowledge base text, not by hoping the AI guesses better.

## Credentials used

- **Google Sheets OAuth2** — append the inbound-message and outcome rows to the Chat Log (×2 nodes)
- **Gmail OAuth2** — send the owner lead/human alert and the AI-failure alert (×2 nodes)
- **OpenAI API key — Header Auth** — the AI reply step (×1 HTTP node)
- **WhatsApp provider API key — Header Auth** — sending the reply through your provider (×1 HTTP node)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only message or email from accounts *you* connect.

## Good to know

- This requires the **official WhatsApp Business Platform** — Meta Cloud API directly (free tier, with limits) or a BSP like Twilio / 360dialog / WATI, which charges per conversation. There is no legitimate free unofficial route, and unofficial gateways risk losing your number.
- You need a **verified business phone number** on the platform before any of this works.
- Business-initiated messages **outside the 24-hour customer-service window** must use pre-approved template messages. This workflow only replies inside a conversation the customer started — it cannot send broadcasts, reminders or cold messages.
- **No long-term memory:** each reply uses only the current message plus the knowledge text in CONFIG. The bot does not remember what the customer said three messages ago.
- Human handoff is an **email alert**, not a live chat takeover — you still pick the conversation up yourself in WhatsApp.
- The AI can still phrase things imperfectly, even with a good knowledge base. Review the Chat Log daily at first.
- If the AI step fails or returns unparseable output, the customer still gets the CONFIG fallback message and the chat is flagged as needing a human — they are never sent raw model output.

## Want the grown-up version?

The RozContent paid build adds: live human takeover inside the same WhatsApp thread, conversation memory so follow-up questions make sense, approved template-message follow-ups and reminders outside the 24-hour window, booking/order integrations, and monitoring so a broken provider connection gets fixed before it costs you customers.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
