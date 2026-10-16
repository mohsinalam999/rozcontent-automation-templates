# Template 27 — YouTube Channel Operator ⭐ FREE TEMPLATE

**This template is FREE** — the complete workflow file is published in this folder, yours to import and run. If you'd rather not wire it up yourself, **RozContent will connect and install it for you for $100 one-time** — set up on a fresh n8n account that *you* create, and the moment we hand over, you change the password and our access ends completely (the full five-step model is at the bottom of this guide), with optional care afterwards.

**The problem:** a faceless YouTube channel is five jobs wearing one hat — finding topics, writing scripts, generating visuals, voicing, editing, uploading, replying, tracking numbers. Each is small; together they eat the week, and the channel quietly stops the first time life gets busy. Hiring it out costs more than a small channel earns.

**What this workflow does:** it operates one channel end to end, on a rhythm you control from a single Google Sheet:

- **Production (fixed time).** On your upload days it takes the next topic from your `Topics` queue — deduped against everything already published, so the channel never repeats itself. Queue empty? It brainstorms five candidates and parks them for your approval instead of inventing a video on its own. For the chosen topic it writes an **original** script, splits it into scenes with a visual prompt each, generates a clip per scene through the **Video Module (Google Flow by default — swappable, see below)**, voices the script with your TTS service, renders the finished video through your render endpoint, generates the thumbnail, writes the full metadata pack (title, description, tags, chapters), and parks the package in the `Ready` tab.
- **Approval gate (default ON).** The finished package — title, thumbnail, preview link, publish time, and a Package ID — lands in your chat. **Nothing uploads until you reply APPROVE.** Only packages you approved (or, in AUTO mode, packages marked ready) can ever publish.
- **Upload (fixed time).** An hourly check publishes the earliest due, approved package: uploads the video with its metadata, sets the custom thumbnail, logs it in the `Videos` tab, stamps the package UPLOADED, and sends you the live link.
- **Chat control.** You run the channel by talking to it: *"kal ka topic badal do"*, *"is hafte upload roko"*, *"stats batao"*, *"APPROVE VID-…"*. An AI agent reads and updates the same sheets everything else runs on — the sheet is always the truth, and anything you edit by hand is respected.
- **Reporting.** Every morning: a stats snapshot for your recent videos into the `Stats` tab, new comments into the `Comments` tab **with a draft reply beside each**, and one digest message. **Drafts are never posted. You approve and post every reply yourself.**

```
PRODUCTION (upload days)                UPLOAD (hourly check)
Schedule ─▶ Config ─▶ Gate ─▶ Topics ─▶ Pick ─▶ Script ─▶ Scenes ─▶ ┌─ VIDEO MODULE ─┐ ─▶ Voice ─▶ Render ─▶ Metadata ─▶ Thumbnail ─▶ Ready tab
   queue empty ─▶ Brainstorm ─▶ Candidates (await your OK)          │ Google Flow    │                                                       │
                                                                    │ (swappable)    │        APPROVAL GATE: package ─▶ your chat ─▶ APPROVE? ─┘
                                                                    └────────────────┘                                                       ▼
                                                                                                          due + approved ─▶ YouTube ─▶ log ─▶ live link to you
CHAT (any time)   You ─▶ WhatsApp / Telegram ─▶ AI Agent (sheet tools) ─▶ reply        REPORTING (daily)   Videos log ─▶ stats ─▶ Stats tab ┐
                                                                                                                             └─▶ comments ─▶ drafts ─▶ digest to you
```

**Who it's for:** owners of small faceless channels (finance explainers, stories, facts, motivation, history — any slideshow/voiceover format) who want the channel to run like a system instead of a chore, while keeping a human hand on the publish button.

**Setup time:** ~60–90 minutes · **Difficulty:** Advanced — this is the catalogue's flagship. Every individual step is guided below, and the $100 install exists precisely because this one has more moving parts than our other templates.

---

## Before you start — what you need

Everything runs on **your own accounts**. The template orchestrates services; it is not one, and their costs are yours.

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs the workflow | Free tier is fine |
| A Google account + one Google Sheet | The Config, queues and logs all live there | Free |
| A YouTube channel + YouTube Data API access | Uploads, stats and comments for **your** channel, via your own OAuth sign-in | Free (API quota is more than enough — see *Good to know*) |
| An OpenAI API key | Scripts, metadata, topic brainstorms, the chat agent | Pay-as-you-go; a video's writing costs cents |
| WhatsApp Business Platform (Cloud API) — **our sample runs on this** | Chat control + all notifications on WhatsApp, via Meta's official API and your own business number | Free tier; Meta pricing after |
| — or a Telegram bot (free, from BotFather) | The same chat control + notifications, on Telegram instead | Free |
| A Google Flow account | The default Video Module — generates each scene's clip (Veo) | Your Flow plan/credits |
| A TTS (text-to-speech) service with an HTTP API | The voiceover, in the voice you choose | Varies by provider |
| An image-generation service with an HTTP API | Thumbnails | Varies by provider |
| A render/assembly endpoint | Stitches clips + voiceover + subtitles into the finished video (contract below) | Yours — a small cloud function or a local bridge both work |

## The sheet — six tabs

Create one Google Sheet with these tabs, named exactly, headers in row 1 in this order:

| Tab | Headers |
|---|---|
| `Config` | `Key` \| `Value` |
| `Topics` | `Topic` \| `Angle` \| `Status` \| `Added` |
| `Ready` | `Package ID` \| `Topic` \| `Title` \| `Description` \| `Tags` \| `Chapters` \| `Video URL` \| `Thumbnail URL` \| `Publish At` \| `Status` \| `YouTube URL` \| `Created At` |
| `Videos` | `Video ID` \| `Topic` \| `Title` \| `YouTube URL` \| `Published At` \| `Source Package` |
| `Stats` | `Date` \| `Video ID` \| `Title` \| `Views` \| `Likes` \| `Comments` |
| `Comments` | `Comment ID` \| `Video ID` \| `Video Title` \| `Author` \| `Comment` \| `Published At` \| `Draft Reply` \| `Status` |

**The `Config` tab is the control panel.** One key per row:

| Key | What it does | Example / default |
|---|---|---|
| `channel_name` | Shown in digests and messages | `Daniel Invests` |
| `channel_id` | Your channel's UC… ID (reporting) | `UC…` |
| `niche` | What the channel is about — drives scripts + brainstorms | `personal finance for beginners` |
| `language` | Script + subtitle language | `English` (default) |
| `voice` | Voice name/ID your TTS service expects | `default` |
| `visual_style` | Style sentence added to every visual prompt | `clean flat vector illustration…` |
| `upload_days` | Which days production runs | `Mon, Wed, Fri` (default) |
| `publish_time` | Time of day videos go live | `18:00` (default) |
| `approval_mode` | `APPROVE` (you approve each video — recommended) or `AUTO` | `APPROVE` (default) |
| `uploads_paused` | `TRUE` stops production AND uploads. The chat agent flips this; so can you | `FALSE` (default) |
| `chat_app` | Which chat app carries control + notifications: `whatsapp` or `telegram` | `whatsapp` (default — our sample) |
| `owner_whatsapp_number` | Your WhatsApp number, full international format — the ONLY number allowed to command or approve on WhatsApp (setup step 5) | `92300…` |
| `owner_telegram_chat_id` | Your Telegram chat ID — the ONLY chat allowed to command or approve on Telegram (setup step 5) | `123456789` |
| `aspect_ratio` | `16:9` long-form or `9:16` Shorts | `16:9` (default) |
| `scene_seconds` | Target seconds per scene | `6` (default) |
| `video_module_endpoint` | URL of your Video Module plug (Flow route by default) | `https://…` |
| `tts_endpoint` | URL of your TTS service | `https://…` |
| `image_endpoint` | URL of your image service (thumbnails) | `https://…` |
| `render_endpoint` | URL of your render/assembly endpoint | `https://…` |
| `digest_recent_days` | Reporting window | `28` (default) |
| `digest_max_videos` | Cap on videos per digest | `10` (default) |

## The Video Module — Google Flow by default, swappable by design

Scene generation is deliberately **one boxed module** in the workflow (see its sticky note). Everything before it hands in scene prompts; everything after it expects clip URLs back. The contract:

**Input — one call per scene:**
```json
{ "scene_number": 1, "prompt": "a kitchen table with a payslip and a phone calculator, morning light",
  "narration": "Most people plan their month after the money is already gone.",
  "aspect_ratio": "16:9", "duration_seconds": 6 }
```
**Output — one response per scene:**
```json
{ "scene_number": 1, "clip_url": "https://…/scene-1.mp4" }
```

**The default plug is Google Flow (Veo)**, running on **your** Flow account: point the module node's URL at your Flow access route, connect your key, and your Flow plan/credits power the generation. Be clear-eyed about the surface: Flow is driven through its own interface/API surface **as available to your account** — this template defines the socket; Flow is the plug that ships in the diagram. Prefer a different generator next month? Replace that **one node's** URL + credential with any service that honours the contract above. Nothing else in the workflow knows or cares which generator made the clips.

## The Chat Channel — WhatsApp or Telegram, your choice

Chat is the second boxed module in this workflow, built on the same socket idea as the Video Module. The agent, the approval gate and every notification never talk to an app directly — outbound messages route through one dispatch step, inbound messages through one normalising step. What a chat channel must provide is a three-line contract:

- **receive** a text message from the owner,
- **send** a text message back to the owner,
- **carry the owner's identity** (their number or chat ID), so the allowlist can check it.

Two channels are implemented, both on official APIs only:

- **WhatsApp** — the official **WhatsApp Business Platform (Cloud API)**, on **your own business number** with your own Meta app credentials. *This is the channel our sample runs on* — `chat_app = whatsapp` ships pre-selected in the example Config.
- **Telegram** — the official **Telegram Bot API**, via your own bot from BotFather. Set `chat_app = telegram`, connect the bot credential, fill `owner_telegram_chat_id` — nothing else changes.

**The allowlist is the lock.** Inbound, both triggers feed the same path — normalise → Config → **Chat Gate** → agent — and the gate lets exactly one identity through: `owner_whatsapp_number` on WhatsApp, `owner_telegram_chat_id` on Telegram. Messages from anyone else are dropped silently, so nobody can borrow approval power by messaging your bot or number. Replies always return on the channel the command arrived on; notifications (approval requests, live links, digests, alerts) go out on the channel `chat_app` selects.

**Honesty lines.** Unofficial WhatsApp automation — tools that drive a *personal* number or a browser session — is **not supported**: it breaks WhatsApp's rules and gets numbers banned, which is why this template speaks only to the official Business Platform. A WhatsApp Business number + Meta app credentials are required for the WhatsApp channel; there is no legitimate way around that, and any tool promising one is selling you the ban risk. **Botim** was examined for this socket and is **not supported**: as of October 2026, Botim offers no official bot or API access for third-party systems. Because the channel is a socket, Botim — or any future app — can be added by writing one new dispatch branch if it ever opens an official API; the agent and the gate will not need to change.

## The render endpoint contract

n8n does not edit video natively, so assembly is one honest HTTP call. Your endpoint (a small cloud function, a hosted renderer, or a script on a machine you control) receives:

```json
{ "render_key": "topic-slug-2026-10-16",
  "clips": [ { "scene": 1, "url": "https://…", "narration": "…" } ],
  "audio_url": "https://…/voiceover.mp3",
  "subtitles": { "burn_in": true, "source": "narration", "language": "English" },
  "aspect_ratio": "16:9", "format": "mp4" }
```
…and returns:
```json
{ "video_url": "https://…/finished.mp4", "duration_seconds": 214 }
```
Subtitles are burned in **from the narration text**, so the words on screen are the words spoken. The TTS call is equally simple — `POST { text, voice, language }` → `{ audio_url }` — and thumbnails — `POST { prompt, aspect_ratio }` → `{ image_url }`.

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the sticky notes left to right.
2. **Build the sheet** (tabs + Config keys, above). Copy its ID from the URL and paste it into every Google Sheets node, replacing `PASTE_YOUR_GOOGLE_SHEET_ID`.
3. **Connect credentials** (all placeholders in the file — click each node and sign in with **your** accounts): Google Sheets, OpenAI (four places: chat model + three HTTP calls — the same key), your chat channel (WhatsApp Business Cloud and/or Telegram bot — connect the one you use, or both), YouTube OAuth, and one header-auth key each for the video module, TTS, image and render endpoints.
4. **Point the four service nodes at your endpoints.** The URLs read from the Config tab (`video_module_endpoint`, `tts_endpoint`, `image_endpoint`, `render_endpoint`) — fill the keys, not the nodes. If your services expect a different request shape than the contracts above, adjust that one node's body; the contracts are the reference.
5. **Set your chat identity (the allowlist).** In `Config`, leave `chat_app` on **`whatsapp`** (the shipped sample) or set it to `telegram`, then fill that channel's owner identity. **WhatsApp:** `owner_whatsapp_number` = the number *you* will message from, in full international format (e.g. `92300…`) — and in your Meta app, point the WhatsApp webhook at the *WhatsApp Trigger* node so your messages reach the workflow. **Telegram:** message your bot once, run the *Telegram Trigger* node in test mode, read `message.chat.id` from the event, and put it in `owner_telegram_chat_id`. Only that identity can command or approve — everything from anyone else is ignored silently. Until this is set, notifications have nowhere to go.
6. **Wire the agent's tools.** The chat agent thinks; three small helper workflows are its hands — *Read Channel Sheets*, *Set Flags & Approvals*, *Manage Topics Queue*. They are imported with the full package (or built for you in the $100 install): paste each helper's workflow ID into its Tool node, replacing `PASTE_HELPER_WORKFLOW_ID`.
7. **Set the clocks.** Two schedule triggers carry clock-times: *Production — Daily Check* (default 09:00) and *Daily Digest — 08:00*. Set them to your times, and set the workflow timezone (⋯ → Settings) to yours. The **days** and the **pause** live in the sheet (chat can change them instantly); clock-times live in the triggers — see *Good to know*.
8. **Test it (worked test — do not skip).**
   - In `Config`: set `upload_days` to include **today**, `approval_mode` = `APPROVE`, `uploads_paused` = `FALSE`, `chat_app` = `whatsapp`.
   - In `Topics`: add one row — a topic you genuinely want — Status `Queued`.
   - On the *Upload to YouTube* node, temporarily set privacy to **unlisted** for this first test.
   - Click **Run Manually (Production)**. Watch the run: script → scenes → module clips → voice → render → metadata → thumbnail → package in `Ready` with Status `AWAITING_APPROVAL`, and an approval message in your chat quoting a Package ID like `VID-20261016-42`.
   - In `Topics`, clear the queue and run production once more: you should get brainstorm candidates stamped *Candidate — needs approval* and **no** video.
   - Reply to the approval message with `APPROVE VID-…` (the exact ID), check the `Ready` row flips to `APPROVED` — then use **Execute Workflow** on the upload branch (or wait for the hourly check): the video uploads **unlisted**, the thumbnail is set, a row lands in `Videos`, the package is stamped `UPLOADED`, and the live link arrives in chat. Watch it end to end.
   - Message it on WhatsApp — `is hafte upload roko`, confirm `uploads_paused` became `TRUE` in the sheet, run production manually — silence. Then `dobara shuru karo` to resume.
   - Set the YouTube node back to **public**, and set `upload_days` back to your real days.
9. **Go live.** Flip the workflow to **Active**. Keep two habits: glance at the approval message before you approve (that glance *is* the product), and skim the morning digest — it is how you notice a broken service before a publish day does.

## The approval gate — and the full-auto trade-off, plainly

With `approval_mode = APPROVE` (the default, and our recommendation), a finished video **cannot** publish until your chat reply approves its Package ID. Rejection bins it — the package is stamped REJECTED and the uploader ignores it forever. The gate costs you one glance per video.

With `approval_mode = AUTO`, packages are stamped READY and publish at their `Publish At` **unreviewed** — including the occasional wrong fact, mangled title, or off-brand frame the pipeline will eventually produce. YouTube judges the *channel* for what it publishes, not the tool that made it. Full-auto is a legitimate choice for a high-volume channel whose owner accepts that risk and reviews published videos after the fact — it is your risk to take, knowingly, which is why the switch is yours and the default is APPROVE.

You can also approve or reject by editing the `Status` cell in the `Ready` tab by hand. **The sheet always wins.**

## Chat control — what you can say

Anything you'd say to an assistant. The agent reads the sheets before acting and confirms what it changed:

- *"kal ka topic badal do — kal wali video savings par banao"* → replaces the next queued topic
- *"is hafte upload roko"* → pauses production + uploads (`uploads_paused = TRUE`)
- *"dobara shuru karo"* → resumes
- *"stats batao"* → answers from `Stats` + `Videos`
- *"jo candidates aye thay, un me se pehla approve kar do"* → queues an approved candidate
- *"APPROVE VID-20261016-42"* / *"REJECT VID-20261016-42"* → the approval gate
- *"approval mode auto kar do"* → flips the mode (it will only do this on your explicit instruction)

The agent **never** posts comment replies and **never** uploads anything itself; publishing only ever happens through the uploader, after approval.

## Credentials used

- **Google Sheets OAuth2** — Config, queues and logs (many nodes)
- **OpenAI API key** — script, brainstorm + metadata calls, and the chat agent's model
- **WhatsApp Business Cloud API** (Meta) — chat trigger, notifications, digests, agent replies on WhatsApp
- **Telegram Bot API** — the same, on Telegram (connect either channel or both; `chat_app` chooses)
- **YouTube OAuth2** — the upload itself, on **your** channel account
- **YouTube Data API key** (header auth) — stats + comments reads
- **Header-auth keys** — your video-module (Flow route), TTS, image and render endpoints

All credentials in the file are placeholders. Nothing is pre-connected; every service signs in as *you*.

## Good to know

- **Why days live in the sheet but times live in triggers:** n8n triggers own clock-times; the sheet owns everything chat needs to change — days, pause, mode. Changing *when in the day* things run means editing two trigger nodes (30 seconds). Changing *which days*, pausing, or switching modes never touches n8n at all.
- **One upload per hourly run, earliest first.** A backlog drains politely instead of bursting five videos onto the channel in one hour.
- **Dedupe is sheet-deep.** Topics are checked against the `Videos` log; comments against the `Comments` tab; packages publish once because publishing stamps them. Re-running any branch by hand is safe.
- **API quota is a non-issue.** Each stats or comments call costs 1 quota unit of your daily 10,000; this template uses a few dozen a day. Uploads cost more per call and are still nowhere near the cap at this cadence.
- **Custom thumbnails need an eligible channel** (YouTube requires the channel to qualify for custom thumbnails). If yours doesn't yet, the upload still succeeds — set thumbnails manually until it does.
- **Check the YouTube node on your n8n version.** Its title/description/tags expressions are pre-filled from the package; if your n8n version renames a field, remap it once during setup — the test upload in step 8 is where you'll see it.

## Honest limits

This template **orchestrates** — it does not conjure. The script model, Flow, the TTS voice, the image service and the render endpoint are yours: their quality is the video's quality, and their bills are yours. The format is the **faceless slideshow/voiceover format** — generated clips + voiceover + burned-in subtitles — unless your render endpoint does more (presenters, screen capture, your own footage are custom-build territory). If a production service fails mid-run, the run stops and alerts you in chat rather than queuing a half-made video — but a service that returns a *bad* result successfully (a garbled voice line, an off-topic clip) can only be caught by your approval glance, which is the strongest argument for leaving the gate on. Chat control understands casual language well, not perfectly: if a command is ambiguous, the agent is instructed to ask one short question rather than guess — and the sheet remains the manual override for everything. Chat itself is official-API only: a personal WhatsApp number driven by unofficial tooling is a ban waiting to happen, so this template refuses that route by design. Finally, the agent's three helper workflows are part of the full package/install: the main workflow file references them by ID, because an agent's tools are separate small workflows in n8n, not nodes in this one.

## Compliance — read this before you switch anything to AUTO

- **Original scripts only.** The writing prompts demand original material and varied hooks/titles (the anti-template rule) — no copied, translated or spun scripts, ever. YouTube's **reused-content policy** is aimed exactly at channels that re-upload or lightly rework other people's material; originality is both the rule and the ranking strategy.
- **AI disclosure stays on.** Videos made this way must carry YouTube's altered/AI-content disclosure — set it on upload (the YouTube node's options / Studio) and keep it on. Hiding AI involvement risks the channel.
- **Monetisation judges the channel.** Watch time and revenue follow YouTube's rules for your channel, not this template's promises — this template makes no earnings claim whatsoever.
- **Comment replies are drafts**, always — bulk auto-replies are spam-shaped, and a wrong auto-reply under your name costs more than the minutes saved.
- **Approval mode is the recommended default** because a human glance is the cheapest compliance department a small channel can have.

## Want it running without the wiring?

The template is free — but if you'd rather be making videos than connecting endpoints, **RozContent will connect and install Template 27 for you for $100 one-time.** It works like this:

1. **You create a new n8n account** — your own, cloud or self-hosted, created fresh for this.
2. **You share that login with us through a secure channel** — never in plain chat or email.
3. **We perform the full setup on your account** — import the template and connect *your* YouTube channel, *your* Flow account and *your* AI keys inside *your* n8n, along with your Sheet, your service endpoints and the chat agent's tools.
4. **We run one full production cycle live** — topic to packaged video, through the approval gate — and show you the system working on your own system.
5. **Handover: you immediately change your n8n login password.** That ends our access completely — and you can verify it yourself: the old password no longer works. From that moment we cannot see or touch your system.

The only route by which access is ever re-granted is the optional care plan ($75/mo), which keeps the system running as services change their APIs — granted by you, at your choice, only if you take it. Custom modules — a different video generator in the swappable slot, a new chat channel if an app ever opens an official API (see the Chat Channel section), a Shorts + long-form pair of channels on one operator — are scoped as custom builds.

**RozContent Automation — done-for-you builds from $250 + care plans. (Template 27: free file, $100 one-time install.)**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://rozcontent.com)
