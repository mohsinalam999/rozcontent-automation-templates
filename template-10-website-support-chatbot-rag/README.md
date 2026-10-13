# Template 10 — Website AI Support Chatbot (RAG)

> ⚠️ **Advanced template — read this first.** This is the most technical template in the pack. It needs **two external services you own and set up yourself**: a paid-tier AI account (OpenAI) and a vector database. The vector-store steps below are Supabase-flavoured placeholders that **will need adapting to your provider (30–60 minutes for a technical user)**. If you are not comfortable creating database tables, pasting API keys into header-auth credentials, and editing HTTP request URLs/bodies, **do not start this one yourself** — take the done-for-you route at the bottom of this page. RozContent sells this exact system as a built-for-you service for a reason.

**The problem:** your website visitors ask the same questions every day — prices, opening hours, delivery, refunds, what's included — and the answers already sit on your FAQ, pricing and policy pages. But visitors don't read pages; they message, wait, and often leave. A generic chatbot is worse than none: it invents prices and policies you never published.

**What this workflow does:** it reads **your own published pages**, breaks them into searchable chunks, and stores them as vectors. When a visitor asks a question in your website chat widget, the workflow finds the closest chunks from your documents and lets AI answer **only from those chunks** — and if the answer isn't in your documents, it returns a safe fallback message ("I don't have that in our published information — please WhatsApp us…") instead of making something up.

```
FLOW A — Ingestion (you run it manually, and re-run it when your pages change)
  Run manually ─▶ CONFIG ─▶ List source pages ─▶ Fetch page ─▶ Clean & chunk (~1,200 chars)
                                                              ─▶ Embed chunk (OpenAI)
                                                                    ├─ OK ────▶ Store vector (your database)
                                                                    └─ Error ─▶ Email alert to you

FLOW B — Website chat (runs on every visitor question)
  Chat widget ─▶ Webhook ─▶ Normalise question ─▶ Embed question ─▶ Search knowledge base (top 4 chunks)
                             (embed or search fails ─▶ Fallback answer, returned to the widget)
                                                              ─▶ Build answer prompt
                                                              ─▶ AI answers ONLY from your documents
                                                              ─▶ Format response ─▶ { answer, sources } back to the widget
```

**Who it's for:** businesses with real FAQ / policy / pricing pages and repetitive support questions — clinics, agencies, e-commerce stores, course sellers, local services with published terms. If your answers are *not* written down anywhere public, this won't help you yet: write the pages first.

**Setup time:** ~60–90 minutes · **Difficulty:** Advanced

---

## Before you start — what you need

| Thing | Why | Cost |
|---|---|---|
| An n8n instance (Cloud or self-hosted) | Runs both flows | Free tier is fine |
| An OpenAI API account with billing enabled | Embeddings (`text-embedding-3-small`) turn text into vectors; `gpt-4o-mini` writes the answers | **Paid, but tiny:** fractions of a cent per chunk and per answer. Budget a few dollars a month at small-business volumes — it is not zero |
| A vector database account — Supabase (pgvector), Pinecone or Qdrant | Stores your document chunks + vectors and performs the similarity search. **This is a separate service you own** — it does not live inside n8n | Supabase free tier works for this; Pinecone/Qdrant also have free/starter tiers with limits |
| Public web pages with your real answers on them | The bot can only ever know what's on these pages | — |
| A Gmail / Google account (optional) | Sends you the ingestion-failure alert | Free |
| A chat widget on your website that can POST JSON to a URL and display the JSON reply | The visitor-facing half. This template is the brain, not the widget | Depends on your site/widget |

## Step-by-step setup

1. **Import.** In n8n: **Workflows → ⋯ → Import from File** → choose `workflow.json`. Read the three sticky notes in the workflow before touching anything else.
2. **Create the vector store (Supabase version).** Create a Supabase project, enable the `vector` (pgvector) extension, and create a table named `documents` with columns `id`, `content` (text), `embedding` (vector(1536) — the size `text-embedding-3-small` returns) and `source_url` (text). Then create a `match_documents` RPC/function that takes a query embedding and a match count and returns the closest rows with their `content` and `source_url`. Supabase's own "vector / embeddings" guides include a ready-made version of this function — use theirs rather than writing one from scratch. **Using Pinecone or Qdrant instead?** The concept is identical, but the two HTTP nodes in step 6 need their URL, auth header and body shape changed to that provider's API.
3. **Create the credentials.** In n8n → **Credentials → Create new → Header Auth**:
   - One named e.g. `OpenAI API key` — Name: `Authorization`, Value: `Bearer YOUR_OPENAI_API_KEY`. Select it in *Embed Chunk*, *Embed Question* and *AI — Answer From Our Documents*.
   - One for your vector store — Name/value per your provider (Supabase expects its API key header). Select it in *Store Vector* and *Search Knowledge Base*.
   - Connect your Google account on *Alert — Ingestion Problem* (Gmail).
   
   Every credential in the imported file is a placeholder dummy; nothing is pre-connected.
4. **Edit the CONFIG node** (in Flow A). Set:
   - `config_business_name` → your business name
   - `config_owner_email` → your email (kept here for reference; see step 7 for the alert's actual recipient)
   - `config_fallback_message` → the exact safe answer for questions your documents don't cover. The shipped default is: *"I don't have that in our published information — please WhatsApp us…"*
   - `config_vector_table` → `documents` (reference value; the table name also lives in the Store Vector URL in step 6)
   - `config_source_pages` → just a pointer note — the actual URLs live in the next step.
5. **List your source pages.** Open *List Source Pages* and replace the three example URLs in the `const URLS = [...]` array with your real public pages (FAQ, pricing, policies, delivery, returns…). One item is produced per URL. Only add pages whose content you are happy for a bot to quote.
6. **Point the vector nodes at your store.** In *Store Vector*, replace `https://YOUR-VECTOR-STORE.example/rest/v1/documents` with your real insert endpoint. In *Search Knowledge Base*, replace `https://YOUR-VECTOR-STORE.example/rest/v1/match_documents` with your real search/RPC endpoint. Check the body shapes while you're there:
   - *Store Vector* sends `{ content, embedding, source_url }`. The embedding is read from `$json.data[0].embedding` — **that field path depends on the embeddings response shape**; if your provider wraps the response differently, adjust it.
   - *Search Knowledge Base* sends `{ query_embedding, match_count: 4 }`.
   
   These two shapes are Supabase-flavoured starting points, not universal standards — adapting them is the expected technical step in this template.
7. **Set the alert recipient — inline.** *Alert — Ingestion Problem* is in Flow A's error branch, so its **Send To is set inline in that node** (`owner@yourbusiness.com`). Replace it there with your real inbox. Flow B likewise has no CONFIG upstream, so the fallback text used by *Fallback Answer* and *Format Response* is written inline in those nodes — if you change `config_fallback_message`, change it in those two nodes too so everything stays identical.
8. **Run ingestion (Flow A).** Click *Run Ingestion Manually* → **Execute workflow**. Each page is fetched, cleaned, chunked (~1,200 characters with overlap), embedded and stored. Afterwards, open your vector table and confirm rows exist with content, an embedding and a source URL. Note: a page that fails to fetch is **skipped silently** — its failure appears in the execution log, so glance at the log after every ingestion run.
9. **Test the chat (Flow B).** With the workflow open, click **Execute workflow**, then POST a question your pages clearly answer:
   ```bash
   curl -X POST https://YOUR-N8N-URL/webhook-test/support-chat \
     -H "Content-Type: application/json" \
     -d '{"question":"What is your refund policy?","session_id":"test-1"}'
   ```
   You should get back JSON like `{"answer":"...","sources":["https://yourbusiness.example/policies"]}`. **Then test the question that matters most:** ask something your documents do **not** answer (your CEO's birthday, a discount you never published). The only acceptable reply is the fallback message. If it invents an answer, do not go live — check that your search step is returning real chunks.
10. **Go live.** Flip the workflow to **Active**, then point your website chat widget at the **Production URL** (`/webhook/support-chat`, without `-test`). The widget should POST `{"question": "...", "session_id": "..."}` (`message` is also accepted instead of `question`) and display the returned `answer`; `sources` can be shown as "Source: …" links under the answer. **Re-run Flow A (step 8) every time your pages change** — until you do, the bot keeps answering from the old text.

## Credentials used

- **OpenAI — Header Auth** (`Authorization: Bearer …`) — embeddings ×2 nodes (*Embed Chunk*, *Embed Question*) and chat completions ×1 (*AI — Answer From Our Documents*)
- **Vector store — Header Auth** — insert chunks (*Store Vector*) and similarity search (*Search Knowledge Base*); credential name/format depends on your provider
- **Gmail OAuth2** — the ingestion-failure alert ×1 (*Alert — Ingestion Problem*)

All credentials in the file are placeholders. Nothing is pre-connected, and the workflow can only act through accounts *you* connect.

## Good to know

- **It is not free to run.** Embeddings and chat calls are fractions of a cent each, but they are metered, and a busy chat widget adds up. The vector database is a separate service you own and are responsible for — Supabase's free tier is genuinely enough to start.
- **The vector-store HTTP shapes WILL need adapting.** Store and search are Supabase-flavoured placeholders. Budget 30–60 minutes of technical fiddling here — this is the step that makes this template Advanced, and the main reason it's offered as a done-for-you build.
- **Ingestion is manual.** There is no schedule watching your pages. Changed a price on Tuesday? The bot still quotes the old one until you re-run Flow A. (Re-running also adds fresh copies of chunks rather than surgically replacing old ones in this free version — if you re-ingest often, clear the `documents` table first, or ask about the paid version's replace-in-place ingestion.)
- **Retrieval quality = document quality.** Thin, vague or contradictory pages produce thin, vague or contradictory answers. The bot is a mirror of your published information, not an improvement on it.
- **No conversation memory.** This free version treats every question on its own — "and how much is *that* one?" has no context. `session_id` is accepted and passed through the normalise step so your widget can send it, but the workflow does not use it to remember anything.
- **Answers are only as safe as the fallback rule.** The AI is instructed to answer only from the retrieved chunks and to return the exact fallback otherwise — but instructions are not guarantees. Do the unanswerable-question test (step 9) with several invented prices, times and policies before going live, and re-test after any prompt change.
- **Failed fetches are quiet.** In Flow A, a page that can't be fetched is skipped and only shows up in the execution log. Check the log after each ingestion run so a missing page doesn't silently mean missing answers.

## Want the grown-up version?

The RozContent paid build adds: a hosted chat widget installed on your site for you, conversation memory (follow-up questions work), scheduled automatic re-ingestion when your pages change (with old chunks replaced, not duplicated), full chat transcripts so you can read what visitors actually ask, human hand-off to WhatsApp mid-chat, and monitoring so a failed embedding or an empty knowledge base gets fixed before your visitors notice. Most owners take this route — the vector-store setup alone usually costs them more in time than the build costs in money.

**RozContent Automation — done-for-you builds from $250 + care plans.**
📱 WhatsApp [+92 312 4207507](https://wa.me/923124207507) · 🌐 [RozContent](https://still-breeze-c782.tayyab-scorpio23.workers.dev)
