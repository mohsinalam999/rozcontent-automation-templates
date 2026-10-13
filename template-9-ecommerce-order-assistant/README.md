# Template 9 — E-commerce Order Assistant

**The problem:** every order needs the same handful of jobs done fast — confirm it to the customer, tell the owner, log it somewhere findable, and later say "it's shipped". Do them by hand and they eat your day; skip them and customers chase *you*. And the carts people abandon an hour ago just sit there, quietly expiring.

**What this workflow does:** it runs two small flows in one file. **Flow A** listens for order webhooks from your store: a new order is saved to a Google Sheet, the customer gets a warm confirmation, and you get an instant alert; a fulfilled order gets a shipping update instead — with a tracking line only if the webhook actually carried one. **Flow B** checks an `Abandoned Carts` sheet tab every hour and sends one friendly checkout reminder for any cart left over an hour, then marks the row so nobody is reminded twice.

```
FLOW A — order events
Store webhook ─▶ Normalise Order ─▶ CONFIG ─▶ Order Fulfilled?
                                                    ├─ No ──▶ Save to Sheet ─▶ Customer confirmation
                                                    │                        └▶ Owner alert
                                                    └─ Yes ─▶ Shipping update email

FLOW B — abandoned carts (hourly)
Schedule ─▶ CONFIG (Flow B) ─▶ Read Abandoned Carts ─▶ Keep carts due a reminder
                                                              └─▶ Reminder email ─▶ Mark Reminder Sent
                                                              └─▶ (none due → workflow quietly stops)
```

**Who it's for:** Shopify and WooCommerce store owners who want order admin and cart reminders handled without a paid app subscription.

**Setup time:** ~30 minutes · **Difficulty:** Medium

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account | The Sheet is your order log and cart list; Gmail sends the emails | Free |
| A Shopify or WooCommerce store where you can create webhooks | Delivers order events to Flow A | Your existing store plan |
| Something that can write abandoned carts into a Google Sheet (a checkout/cart app or plugin export) — Flow B only | Flow B reads carts from the sheet; it cannot see carts that never reach it | Some apps/plugins are free, some paid — check yours |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Open it and read the sticky notes left to right. Note there are **two separate flows** in the one file — Flow A (top) and Flow B (bottom) — and they are set up independently.
2. **Create the Google Sheet.** Make a new sheet (any name) with **two tabs**, named exactly:
   - `Orders` with headers in row 1: `Timestamp | Order ID | Customer | Email | Total | Currency | Items | Event`
   - `Abandoned Carts` with headers in row 1: `Timestamp | Customer Name | Email | Cart Total | Currency | Checkout URL | Reminder Sent`

   Copy the sheet's ID from its URL (the long string between `/d/` and `/edit`) and paste it into all three Google Sheets nodes, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect your Google account.** Click the first Google Sheets node → credential dropdown → **Create new** → sign in with your Google account. Repeat on the Gmail nodes. **Use the store's real support inbox** for Gmail if you can — confirmation and shipping emails send from the connected account, and customer replies should land somewhere monitored.
4. **Edit BOTH CONFIG nodes.** In Flow A's `CONFIG — Edit Me` set:
   - `config_store_name` → your store name (used in every email)
   - `config_owner_email` → the inbox that should get new-order alerts
   - `config_support_email` → the support address quoted in customer emails
   - `config_currency` → e.g. `USD` (used when an order payload carries no currency)

   Flow B is self-contained (the schedule flow never passes through Flow A's CONFIG), so set the same store name and support email again in `CONFIG — Store (Flow B)`, and keep the two in sync if they ever change.
5. **Create the store webhook (Flow A).** Open the *Store Order Webhook* node → copy the **Production URL**, then:
   - **WooCommerce:** in your store admin go to **Settings → Advanced → Webhooks → Add webhook** — set Status to Active, Topic to **Order created** (add a second webhook with Topic **Order updated** if you want fulfilment events too), and Delivery URL to the Production URL.
   - **Shopify:** in your store admin go to **Settings → Notifications → Webhooks → Create webhook** — Event **Order creation**, Format JSON, URL = the Production URL. Create a second webhook with Event **Order fulfilment** for shipping updates.
6. **Remap `Normalise Order` with a real payload (expect ~10 minutes).** Payload shapes genuinely differ between Shopify and WooCommerce — field names, nesting (`billing` vs `customer` vs `billing_address`), and status values are not the same. The node ships with defensive fallback expressions (`body.billing… || body.customer…`, `fulfillment_status || status === 'completed'`) that cover the common shapes, but open a test execution (step 8), look at what your store actually sent, and adjust the four key expressions — `order_id`, `customer_email`, `total`, and `event_type` — if any came through empty or wrong. The fulfilment rule as shipped is: an order counts as **fulfilled** if the payload has a `fulfillment_status`, or its `status` is `completed` / `fulfilled`; anything else is treated as a **new order**.
7. **Get abandoned carts into the sheet (Flow B).** This flow only works if abandoned carts actually reach the `Abandoned Carts` tab. Options: a Shopify or WooCommerce checkout/cart-recovery app or plugin that can export or sync carts to Google Sheets, or a manual/plugin export you run regularly. Be honest with yourself here: **many basic stores can't send cart webhooks at all**, so if nothing is writing that tab, Flow B will simply find nothing to do. (The paid RozContent build skips the sheet and reads the store API directly.)
8. **Test it.**
   - *Flow A, new order:* place a real test order in your store (or use the store's test mode) and watch the execution. Check: a row in `Orders`, a confirmation email in the customer's inbox, and a new-order alert in the owner inbox. A minimal manual test payload also works — POST this to the webhook's **Test URL** while listening:
     ```bash
     curl -X POST https://YOUR-N8N-URL/webhook-test/store-order \
       -H "Content-Type: application/json" \
       -d '{"id":1001,"status":"processing","billing":{"first_name":"Test","last_name":"Buyer","email":"you@youremail.com"},"total":"49.00","currency":"USD","line_items":[{"name":"Test product","quantity":1}]}'
     ```
     A Shopify-shaped fulfilment test looks different — note `fulfillment_status` and a top-level `email`:
     ```bash
     curl -X POST https://YOUR-N8N-URL/webhook-test/store-order \
       -H "Content-Type: application/json" \
       -d '{"id":5002,"order_number":1002,"email":"you@youremail.com","customer":{"first_name":"Test","last_name":"Buyer"},"total_price":"49.00","currency":"USD","fulfillment_status":"fulfilled","line_items":[{"title":"Test product","quantity":1}]}'
     ```
     That second payload should take the **fulfilled** branch and produce a shipping-update email (Flow A only writes new orders to the `Orders` sheet). It carries no tracking fields, so the email honestly says tracking details will follow from the courier — if your real fulfilment payload includes `tracking_number` / `tracking_url` fields under those names, remap them in `Normalise Order` and the email will include them.
   - *Flow B:* add a fake row to `Abandoned Carts` with a `Timestamp` more than an hour ago (ISO format, e.g. `2026-10-13T08:00:00Z`), your own email, a cart total and a checkout URL, and leave `Reminder Sent` empty. Run the workflow manually (or wait for the hourly trigger) and check: one reminder email arrives, and the row's `Reminder Sent` is now filled in — run it again and confirm no second email sends.
9. **Go live.** Flip the workflow to **Active** (top-right toggle) and make sure the store webhooks point at the **Production URL** (`/webhook/store-order`, without `-test`).

## Credentials used

- **Google Sheets OAuth2** — append orders, read abandoned carts, mark reminders sent (×3 nodes)
- **Gmail OAuth2** — send the confirmation, owner alert, shipping update and cart reminder (×4 nodes)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only send from an account *you* connect.

## Good to know

- **Payload field names differ between Shopify, WooCommerce and other platforms** — this is the one real setup chore. Budget the ~10-minute remap in `Normalise Order` (step 6) using a test payload from your own store; an order with an empty email or total in the sheet is the tell-tale sign a field needs remapping.
- **Flow B is only as good as the sheet.** If abandoned carts don't reach the `Abandoned Carts` tab, no reminders send — the workflow fails silently in the sense that it correctly finds nothing due. Check the tab has fresh rows before blaming the automation.
- **No tracking is ever invented.** The shipping email includes a tracking number/link only when the payload carried one; otherwise it says tracking details will follow from the courier.
- **One cart reminder per row.** Once `Reminder Sent` is filled, that cart is never reminded again by this free version — no multi-step recovery sequence, no discount codes.
- **Duplicate webhooks create duplicate rows.** Stores sometimes retry a webhook; a quick scan of the `Orders` tab catches duplicates at small-store volumes.
- The hourly check reads the whole `Abandoned Carts` tab each run. Fine for small stores; a very large cart list belongs in a database, not a sheet.

## Want the grown-up version?

The RozContent paid build adds: direct store-API cart sync (abandoned carts pulled from Shopify/WooCommerce themselves — no sheet in the middle, no missing carts), real courier tracking links in the shipping email, a second and third cart-reminder stage with an optional discount code, refund and returns notification flows, and monitoring so a broken webhook gets fixed before orders pile up unconfirmed.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
