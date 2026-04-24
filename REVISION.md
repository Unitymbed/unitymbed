# UnityMbed — Revision History

AI firmware assistant for ARM Cortex MCUs (Nations N32 series).
Full-stack: CLI + Firebase backend + admin web app + managed knowledge base.

> This document chronicles every public release from `v0.1.0` to `v0.5.4`.

---

## Project scale snapshot (v0.5.4, 2026-04-25)

| Metric | Value |
|---|---|
| **Public releases** | 27 (`v0.1.0` → `v0.5.4`) |
| **Total source LOC** | 7,497 (TypeScript/TSX) |
| &nbsp;&nbsp;`apps/cli/src` | 3,957 lines (49 files) |
| &nbsp;&nbsp;`apps/functions/src` | 1,875 lines (21 files) |
| &nbsp;&nbsp;`apps/functions/scripts` | 440 lines (5 files) |
| &nbsp;&nbsp;`apps/admin/src` | 1,225 lines (12 files) |
| **Cloud Functions deployed** | 23 |
| **Firebase Hosting sites** | 1 (`unitymbed-admin.web.app`) |
| **Knowledge base chunks** | 7,027 (N32G031 + N32G452 + N32G455) |
| **Stripe Payment Links** | 3 (Start / Pro / Advance, Live) |
| **Supported platforms** | macOS arm64/x64, Linux x64, Windows x64 |
| **Distribution** | Homebrew tap + GitHub releases |
| **Estimated build time** | ~75–90 hours over ~4 days of focused work |

---

## Architecture (current state)

```
┌──────────────────────┐      ┌──────────────────────────────────┐
│ Customer CLI (brew)  │      │  Admin Web GUI                   │
│ unitymbed v0.5.4     │      │  https://unitymbed-admin.web.app │
│  • TUI chat + panel  │      │   /docs   /upload  /test         │
│  • /plan /apply      │      │   /billing /admins               │
│  • /undo /history    │      │   /welcome (public, post-pay)    │
│  • build/flash/debug │      └──────────────┬───────────────────┘
│  • token metering    │                     │
└──────────┬───────────┘                     │
           │  SSE (credits-based)            │ Firebase ID token
           ▼                                 ▼
┌──────────────────────────────────────────────────────────────────┐
│ Firebase (enterpriseunitymbed)                                   │
│ ┌────────────┐ ┌─────────────┐ ┌──────────────┐ ┌──────────────┐│
│ │ aiProxy    │ │ kbProcess   │ │ KB admin     │ │ billing      ││
│ │  +RAG      │ │ (Storage    │ │  List/Delete │ │  Get/Set     ││
│ │  +token    │ │  trigger,   │ │  TestQuery   │ │  Stats       ││
│ │   meter    │ │  asia-se1)  │ │  UploadUrl   │ │              ││
│ └────────────┘ └─────────────┘ └──────────────┘ └──────────────┘│
│ ┌────────────┐ ┌─────────────┐ ┌──────────────┐ ┌──────────────┐│
│ │ Stripe     │ │ onboard     │ │ admin ACL    │ │ license mgmt ││
│ │  webhook   │ │  Status     │ │  List/Add/   │ │ (active,     ││
│ │ (+credits  │ │  (public,   │ │   Remove     │ │  quota,      ││
│ │  +renew)   │ │   post-pay) │ │              │ │  checkQuota) ││
│ └────────────┘ └─────────────┘ └──────────────┘ └──────────────┘│
│ ┌──────────────────────────────────────────────────────────────┐ │
│ │ Firestore                                                    │ │
│ │  licenses/  customers/  admins/  kb_docs/  kb_chunks/        │ │
│ │  billing/config  (tier quotas + packs, admin-editable)       │ │
│ │              (vector index: COSINE flat, 768 dim)            │ │
│ └──────────────────────────────────────────────────────────────┘ │
│ ┌───────────────┐ ┌──────────────┐ ┌──────────────────────────┐ │
│ │ Stripe (Live) │ │ Resend       │ │ Secret Manager           │ │
│ │ 3 products    │ │ email        │ │  GEMINI / STRIPE / RESEND│ │
│ │ 3 Payment     │ │              │ │                          │ │
│ │ Links         │ │              │ │                          │ │
│ └───────────────┘ └──────────────┘ └──────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

---

## Cloud Functions inventory (23)

| Function | Purpose |
|---|---|
| `onUserCreate` | Firebase Auth trigger — provision customer doc |
| `activateLicense` | Bind a license key to a machine ID |
| `checkLicense` | Periodic refresh from CLI |
| `checkQuota` | License validity check (aiProxy does real enforcement) |
| `getUsage` | Usage summary — credits, tokens, requests per period |
| `createFreeLicense` | `unitymbed signup` backend |
| `stripeWebhook` | Subscription lifecycle + renewal + credit purchase + emails |
| `buyCredits` | Stripe Checkout for credit packs (reads pack catalog from config) |
| `onboardStatus` | **public** — post-Payment-Link polling by session ID |
| `syncUpload` / `syncDownload` | Project cloud backup |
| `aiProxy` | **Core AI path** — SSE, license+quota+identity+RAG, Gemini proxy, token meter |
| `adminList` / `adminAdd` / `adminRemove` | Admin allowlist management |
| `kbList` / `kbDelete` / `kbTestQuery` / `kbUploadUrl` | KB admin API |
| `kbProcess` | **Storage trigger** (`asia-southeast1`) — parse PDF → chunk → embed → write |
| `configGet` / `configSet` / `configStats` | Billing config admin API (live pricing edits) |

---

# Version-by-version history

All dates in UTC. Every release is tagged on `Unitymbed/homebrew-tap` with
pre-built binaries for macOS (arm64 + x64), Linux x64, and Windows x64.

---

## `v0.1.0` — 2026-04-21  *Initial public release*
First Homebrew-distributed build. Three BYOK AI backends: Ollama, Gemini, Claude.
TUI with streaming, `unitymbed init` scaffolding, ARM GCC + OpenOCD pipeline.

## `v0.1.1` — 2026-04-21  *Brand labels*
Status bar renames to "Unitymbed AI Cloud / Local".

## `v0.1.2` — 2026-04-21  *Tier naming*
- Claude → **Unitymbed AI Cloud Pro**
- Gemini → **Unitymbed AI Cloud**
- Ollama → **Unitymbed AI Local**

## `v0.1.3` — 2026-04-21  *Usage quota + `unitymbed usage`*
Per-tier monthly caps (Free / Start / Pro / Advance), first version of the quota panel.

## `v0.1.4` — 2026-04-21  *Credits + Stripe Checkout*
`unitymbed credits list / buy` with `cloud_100`, `cloudPro_100` packs.

## `v0.1.5` — 2026-04-21  *License renewal + auto-refresh*
Webhook handles `invoice.paid` (extend 31d) and `customer.subscription.deleted`.

## `v0.1.6` — 2026-04-21  *Quota rebalance, Cloud Pro paused*
Start 200 / Pro 1,500 / Advance 8,000 req/mo. Claude tier disabled.

## `v0.1.7` — 2026-04-21  *Single-provider Cloud*
All tiers route to Gemini.

## `v0.1.8` — 2026-04-21  *Free signup flow*
`unitymbed signup <email>` → zero-payment 20 req/mo license + Resend email.

## `v0.1.9` — 2026-04-21  *N32G031 flash works* 🎉
**First public OpenOCD flash driver for Nations N32G031.** Custom TCL programmer
writes directly to the flash controller at `0x40022000` (unlock keys
`0x45670123` / `0xCDEF89AB`), bypassing `stm32f1x` which fails on the Nations
DBGMCU location at `0x1FFFF508`. End-to-end `init → build → flash` verified on
real hardware.

## `v0.2.0` — 2026-04-21  *Production Ready*
Full N32G031 register reference card embedded in system prompt. Datasheet repo
published. Stripe Live pipeline end-to-end verified with a real ฿249
transaction.

## `v0.2.1` — 2026-04-21  *Memory & Context*
`.unitymbed/conversations/` persists chat history per project. Auto-loads on
reopen. Injects `src/*`, `unitymbed.json`, `CONTEXT.md`. `/new` command.

## `v0.2.2` — 2026-04-22  *Hardware Inspection*
`/check` — AI picks MMIO registers, OpenOCD halts + reads, AI diagnoses
mismatches (suggests only, never auto-edits). `/monitor` — auto-detects
CMSIS-DAP VCP, opens 115,200-baud terminal.

## `v0.2.3` — 2026-04-22  *Auto-update Notifier*
24h GitHub poll, hint in header if new release available. `unitymbed --version`.

## `v0.2.4` — 2026-04-22  *Agent + Thai Input*
- `unitymbed build debug` compiles with `-O0 -g3` for GDB
- `unitymbed agent "<problem>"` — 4-step loop (build debug → flash → register
  snapshot → AI diagnosis)
- `/remember <fact>` persists to `CONTEXT.md`
- Thai input: preview above box when Thai detected, hide cursor to avoid
  combining-mark column misalignment

## `v0.2.5` — 2026-04-22  */agent inside TUI*
Agent loop runs inline as tool-result messages.

## `v0.2.6` — 2026-04-22  *File Operations in TUI*
`/ls [dir]`, `/view <path>`, `/edit <path>` (opens `$EDITOR`).

## `v0.2.7` — 2026-04-22  *Branded Identity*
AI introduces itself as "UnityMbed AI" — never mentions Google / Anthropic / etc.

## `v0.3.0` — 2026-04-22  *Unitymbed AI Cloud (no BYOK)*
Cloud AI proxied through UnityMbed backend only. Users no longer configure
API keys.
- New `aiProxy` Cloud Function with SSE + license + machine + quota
- Identity rule moved server-side so direct callers can't bypass
- CLI removes `@anthropic-ai/sdk` + `@google/generative-ai`; adds
  `UnityMbedCloudClient`
- `config.migrate()` auto-strips legacy `keys.*` / `apiKey`, remaps
  `gemini` / `claude` → `cloud`
- Docs repo rewritten: no BYOK, no Cloud Pro mentions

## `v0.3.1` — 2026-04-23  *RAG Grounded Answers*
Every Cloud query grounds in a curated knowledge base.
- `ai/rag.ts` — Gemini `gemini-embedding-001` @ 768-dim Matryoshka truncation,
  Firestore native `findNearest` (COSINE), fail-open on error
- `aiProxy` accepts `mcu` / `useKB` / `topK`, retrieves top-5 chunks, prepends
  as GROUNDED REFERENCES block, emits `ragHits` SSE frame
- Local ingestion tool at `apps/functions/scripts/ingest.ts` with PDF + code
  chunkers (pdfjs-dist legacy build, no worker; C/H boundary-aware)
- **KB at launch** (N32G031): datasheet 148 + UM 958 + Nations SDK 1,235 =
  **2,341 chunks**

## `v0.3.2` — 2026-04-23  *Hotfix: Cloud default for fresh installs*
`DEFAULT_CONFIG.ai.provider` flipped from `ollama` → `cloud`. `activate`
auto-switches stale configs. Router prefers Cloud when license is present.
Header shows `license.email`.

## `v0.3.3` — 2026-04-23  *Register-name safety + thinking off*
Users reported AI sometimes returned cut-off responses or STM32-style register
names. Three fixes:
- **Disable Gemini 2.5 thinking mode** (`thinkingConfig: { thinkingBudget: 0 }`) —
  thinking tokens were silently eating the output budget
- Raise default `maxOutputTokens` 4096 → 8192
- **REGISTER-NAME SANITY** section in the system prompt mapping common
  mistakes: `MODER→PMODE`, `OTYPER→POTYPE`, `OSPEEDR→SR`, `PUPDR→PUPD`,
  `IDR→PID`, `ODR→POD`, `BSRR→PBSC`
- Explicit "DO NOT regenerate `n32g031.h`" guard in the system prompt — the
  template header is correct; AI was wastefully rewriting it
- RAG output formatter marks `code_example` chunks as "reference only" so AI
  reads the Nations SDK for offsets but doesn't emit HAL calls

## `v0.3.4` — 2026-04-23  *Unified usage/credits + volume packs*
`unitymbed usage` and `unitymbed credits list` were showing incompatible info
(usage said "200/200" while credits list reported 107 available).
- `usage` shows credits column whenever credits > 0, with next-step guidance
  when exhausted
- `credits list` adds a unified account summary + pack table with per-request
  price and % off vs baseline
- New packs: `cloud_500` (20% off), `cloud_2000` (40% off), `cloud_10000`
  (60% off)

## `v0.3.5` — 2026-04-23  *Clearer "drawing from credits" footer*
Footer said "⚠ 0 Cloud AI requests left" even when pack credits were
available. Now distinguishes three cases:
- Monthly cap reached + credits available → informational "drawing from pack
  credits"
- Cap + zero credits → blocking warning with concrete next steps
- Near-cap (≤5 left) → unchanged gentle heads-up

## `v0.4.0` — 2026-04-23  *Token-based billing + admin pricing console*
Replaced "1 request = 1 credit" with **"1 credit = 1,000 tokens"** (input +
output combined). Fair pricing: small questions cost 0.5 credit, full code
generation 5–20 credits.

**Backend:**
- `billing/config.ts` — new Firestore singleton `billing/config` with tier
  quotas, pack catalog, upstream cost assumptions (Gemini $/M), USD→THB rate.
  60-second in-memory cache.
- `aiProxy` — captures Gemini's `usageMetadata.promptTokenCount` +
  `candidatesTokenCount`, computes `credits_used = (in + out) / tokens_per_credit`,
  deducts from monthly first then overflow to pack. Rich `done` frame with
  `tokens`, `credits`, `monthly`, `pack`.
- `getUsage` returns `{monthly, pack, tokens, requests}` (+ legacy block for
  older CLIs).
- `adminConfig.ts` — `configGet` / `configSet` / `configStats` Cloud
  Functions, admin-gated.
- `migrate-v04.ts` script — seeds `billing/config` with defaults, resets
  in-flight per-period usage docs, archives legacy counters under `_legacy`,
  carries over pack credits.

**Defaults:**
| Tier | Monthly credits | Monthly price |
|---|---|---|
| Free     |    200 | ฿0 |
| Start    |  2,000 | ฿249 |
| Pro      | 10,000 | ฿499 |
| Advance  | 50,000 | ฿1,200 |

| Pack       | Credits | Price  | ฿/credit | Discount |
|------------|---------|--------|----------|----------|
| cloud_1k   |  1,000  | ฿249   | 0.249    | — |
| cloud_5k   |  5,000  | ฿999   | 0.200    | 20% |
| cloud_20k  | 20,000  | ฿2,999 | 0.150    | 40% |
| cloud_100k |100,000  | ฿9,999 | 0.100    | 60% |

**CLI:**
- Per-response footer: `📊 4.55 credits  (4,550 in + 4 out = 4.6k tokens)`
- `usage` shows monthly credits, pack credits, token breakdown, guidance
- `credits list` shows per-credit price and % off

**Admin GUI:**
- New `/billing` page — edit tiers, packs, upstream costs, USD→THB rate live
- Live margin calculation per pack + tier worst-case cost
- Stats card: tokens this month, upstream cost, revenue proxy

## `v0.5.0` — 2026-04-24  *Coding Agent TUI*
Major TUI upgrade: firmware-focused coding agent with plan/apply/undo and
session state that survives restarts.

**Session state** (per-project `.unitymbed/session.json`):
- `goal`, `plan[]`, `files[]`, `schemas[]`, `diagrams[]`, `tokens`, `credits`, `messageCount`

**AI output format** (parsed server-side):
- `<unitymbed:plan>` list of tasks
- `<unitymbed:task-start/>` and `<unitymbed:task-done/>`
- `<unitymbed:file path="…">` full file content (queued, not auto-written)
- `<unitymbed:schema name="…">` struct definitions
- `<unitymbed:diagram title="…">` ASCII wiring / flow

**New slash commands:**
- `/plan <goal>` — AI breaks goal into numbered tasks
- `/continue` — pick next pending task
- `/apply` — write ALL queued file edits
- `/discard` — drop pending edits
- `/done <id>` — mark task done manually
- `/status` — print current session
- `/reset` — clear session
- `/panel` — toggle side-panel visibility

**Right-side panel** (visible when terminal ≥100 cols):
- 📋 Todo with progress icons
- 📁 Pending + applied file changes (colour-coded)
- 📐 Schemas · 🔀 Diagrams
- ⏱ elapsed · 💳 credits used · 📍 project & MCU

File edits no longer auto-write when AI uses structured blocks — they queue as
pending so you can review, then `/apply` in batch. Plain markdown code blocks
still auto-write for backward compat.

## `v0.5.1` — 2026-04-24  *Hotfix: parser drops queued files*
Symptoms from a user report:
```
→ structured changes queued — type /apply to write or /discard to drop
❯ /apply
→ no pending changes
```
Two bugs:
1. `parseAiOutput` regex required both opening and closing `</unitymbed:file>`
   tags. Gemini sometimes truncated mid-stream or wrote attributes in a
   different order → parser missed the block while the "queued" message still
   fired from a naive substring test.
2. Session credits didn't advance because the stream-end event didn't carry
   `usageMetadata`.

Fixes:
- Rewrote extractor to scan for open tag, accept any attribute order, recover
  content up to the next `<unitymbed:` tag or EOF when the closing tag is
  missing.
- Dispatcher now prints the actual list of queued files for sanity.
- Ingestor scrapes the `📊 X credits (N in + M out)` footer line to track
  tokens reliably.

## `v0.5.3` — 2026-04-24  *Panel cleanup + project guards*
(v0.5.2 was cut internally but rolled into v0.5.3 before publishing.)

User feedback: right-side panel was rendering a vertical border line all the
way down the terminal height; credit-to-฿ conversion wasn't wanted.

Plus bugs when running the TUI outside a project:
- `/apply` said "no pending changes" even when AI had queued files
- `/fix` crashed with `null is not an object (project.root)`
- Header silently showed `project=undefined`

Fixes:
- Session panel: dropped borders, uses padding instead; no more ฿ in the
  footer
- `/apply`: distinct message when pending exists but no project, with clear
  instruction to `cd` or `unitymbed init`
- `/fix`: explicit project guard
- Header: yellow `⚠ no project (cd into unitymbed.json)` when cwd has no
  `unitymbed.json`
- SessionPanel empty-state shows copy-paste `unitymbed init` commands

## `v0.5.4` — 2026-04-24  */undo /history /restore*
Every `/apply` now creates a timestamped backup under
`.unitymbed/backups/<epoch>/` before overwriting. Three new commands:

| Command | What it does |
|---|---|
| `/undo` | revert the most recent `/apply` |
| `/history` | list backups (newest first) with timestamps |
| `/restore <id>` | revert all the way back to a specific backup |

Implementation:
- `SessionState` gains a `backups[]` array persisted to `session.json`
- For each file in an `/apply` batch, we copy the original (if it existed)
  to the backup directory. Created files get `backupPath: null` and are
  deleted on undo.
- Restored files come back to the pending queue so you can tweak and
  re-apply.
- Graceful migration for sessions saved under v0.5.3 (empty `backups`).

---

# Beyond the CLI — companion deliverables this cycle

## Admin Web GUI — `unitymbed-admin.web.app`
React + Vite + TailwindCSS + shadcn primitives. Six pages:
- **Documents** (/docs) — list, filter by MCU, delete (cascades chunks)
- **Upload** (/upload) — drop PDF → signed Storage URL → `kbProcess` ingests
- **Test Query** (/test) — debug RAG retrieval
- **Billing** (/billing) — edit tiers, packs, upstream costs live
- **Admins** (/admins) — add / remove admin emails
- **Welcome** (/welcome) — **public**, post-Payment-Link polling UI

Auth: Firebase Google sign-in + allowlist in Firestore `admins/{email}`.
Welcome route bypasses auth (customers don't have accounts yet).

## Stripe Payment Links (Live)
Three direct-pay URLs so customers can subscribe without the CLI:

| Plan | Monthly | Link |
|---|---|---|
| Start   | ฿249   | `buy.stripe.com/cNi14gf5Ebn93tE2vOew801` |
| Pro     | ฿499   | `buy.stripe.com/6oU9AM5v43UH9S29Ygew802` |
| Advance | ฿1,200 | `buy.stripe.com/9B614gaPo3UH5BM1rKew803` |

Payment success URL points to `/welcome?session={CHECKOUT_SESSION_ID}`. The
welcome page polls `onboardStatus` every 2.5s until the Stripe webhook
provisions the license, then shows the key + a one-line install command:
```
brew tap Unitymbed/tap && brew install unitymbed && unitymbed activate UM-START-...
```
Zero manual activation; click the copy button and paste into Terminal.

## KB expansion (N32G452 + N32G455)
Added the entire **Nations N32G45x Motor Kit firmware** (Std Periph Driver +
FOC + Motor App) as code_example chunks, plus all 5 Motor Kit User Guide PDFs
as app_notes. Double-tagged under both `n32g452` and `n32g455` since they
share a family SDK.

| MCU | Docs | Chunks |
|---|---|---|
| N32G031 | 3  | 2,341 (DS + UM + SDK) |
| N32G452 | 6  | 2,343 (SDK + 5 UG PDFs) |
| N32G455 | 6  | 2,343 (SDK + 5 UG PDFs) |
| **Total** | **15** | **7,027** |

Datasheets and Reference Manuals for N32G452/G455 still pending — once
uploaded they'll close the register-level gap for those families.

## Documentation repo — `Unitymbed/unitymbed`
- README rewritten for token-based pricing + Payment Links
- REVISION.md (this file) chronicles every release
- `docs/pricing.md`, `docs/ai.md`, `docs/commands.md` kept in sync

---

# Complexity highlights

| Subsystem | ⭐ | Notes |
|---|---|---|
| **Custom OpenOCD N32G031 flash driver** | ⭐⭐⭐⭐⭐ | Reverse-engineered Nations flash controller from vendor SDK; custom TCL unlock sequence; first-of-its-kind in the public ecosystem |
| **Token-based billing with live admin edit** | ⭐⭐⭐⭐⭐ | Real-time Gemini `usageMetadata` → credit deduction → monthly/pack cascade; admin GUI edits pricing with 60s cache invalidation; margin calculator is live in the UI |
| **Stripe → license → email pipeline** | ⭐⭐⭐⭐ | 4 webhook event types × 3 tiers × email delivery × credit metadata branching × signature verification × Payment Link success-URL post-pay onboarding |
| **RAG retrieval inline inside the SSE request** | ⭐⭐⭐⭐ | Embed query → Firestore vector `findNearest` → context assembly → fail-open, all inside the generation round-trip without adding latency |
| **Coding Agent TUI** | ⭐⭐⭐⭐ | Structured-block parser with permissive fallback for partial streams, session persistence, pending-queue UX, backup/undo/restore, layout math that collapses under narrow terminals |
| **Admin GUI with Firebase Auth + allowlist** | ⭐⭐⭐ | ID-token verification → Firestore admin check → CORS allowlist → signed-URL upload → Storage trigger chain |
| **Ingestion pipeline** | ⭐⭐⭐ | pdfjs-dist legacy build (no worker), regex-based C/H function-boundary chunker, concurrent embedding with backoff |
| **Config migration (BYOK → managed; per-request → token)** | ⭐⭐⭐ | On-disk config rewrite + server-side usage doc rewrite with legacy archival |
| **Thai input rendering** | ⭐⭐ | Combining marks misalign terminal columns — preview-above-input + cursor-hidden fixes it |
| **Cross-platform binary build** | ⭐⭐ | `bun build --compile --target=bun-{mac-arm64,mac-x64,linux-x64,windows-x64}` |

---

# External integrations

| Service | Usage |
|---|---|
| **Stripe (Live)** | 3 subscription products (Start/Pro/Advance), 3 Payment Links, dynamic `price_data` for credit packs |
| **Resend** | Transactional email (license issued, renewal, credit receipt). Still on `onboarding@resend.dev` pending DNS verify for `noreply@unitymbed.com` |
| **Firebase Auth** | Admin Google sign-in + server-side ID-token verification |
| **Firestore** | 6 top-level collections — `licenses`, `customers`, `admins`, `kb_docs`, `kb_chunks`, `billing/config` (singleton). Vector index on `kb_chunks.embedding`. Composite index on `licenses(email, createdAt)`. |
| **Firebase Storage** | `kb/{docId}/` admin uploads (signed-URL write, Functions-only read) |
| **GCP Secret Manager** | `GEMINI_API_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY` |
| **Gemini API** | `gemini-2.5-flash` for generation (thinking disabled), `gemini-embedding-001` @ 768-dim Matryoshka for embeddings |
| **GCP Service Accounts** | `kb-ingest@…` for local admin ingestion (Firestore write + Firebase Admin role) |
| **Homebrew tap** | `Unitymbed/homebrew-tap` — formula auto-updated each release |
| **GitHub Releases** | 27 tagged releases with pre-built binaries (macOS arm64/x64, Linux x64, Windows x64) |

---

# Deployed URLs

| Resource | URL |
|---|---|
| AI Cloud proxy | `https://us-central1-enterpriseunitymbed.cloudfunctions.net/aiProxy` |
| Post-payment welcome | `https://unitymbed-admin.web.app/welcome?session=…` |
| Admin web app | `https://unitymbed-admin.web.app` |
| Payment Link — Start | `https://buy.stripe.com/cNi14gf5Ebn93tE2vOew801` |
| Payment Link — Pro | `https://buy.stripe.com/6oU9AM5v43UH9S29Ygew802` |
| Payment Link — Advance | `https://buy.stripe.com/9B614gaPo3UH5BM1rKew803` |
| Main docs repo | `https://github.com/Unitymbed/unitymbed` |
| Homebrew tap | `https://github.com/Unitymbed/homebrew-tap` |
| Templates | `https://github.com/Unitymbed/templates` |
| Sensor library | `https://github.com/Unitymbed/sensor-library` |
| Datasheets | `https://github.com/Unitymbed/datasheets` |

---

# Open items

- **N32G452 + N32G455 datasheets and Reference Manuals** — SDK + UG ingested, but
  chip-level register-by-register spec PDFs still needed for fully-grounded
  bare-metal generation on those parts
- **Gemini key rotation** — the Gemini API key used in Secret Manager was
  shared via chat during setup; rotate at https://aistudio.google.com/app/apikey
  → update Secret Manager → redeploy `aiProxy` / `kbProcess`
- **Resend DNS verification** — switch sender from `onboarding@resend.dev` to
  `noreply@unitymbed.com`
- **Email template with welcome URL** — include `unitymbed-admin.web.app/welcome`
  link in the license email so users who miss the Stripe redirect can still
  land on the onboarding page
- **API Keys for UnityMbed apps** — design exists (Firestore `api_keys`,
  `X-UnityMbed-Key` header on aiProxy, admin GUI page); not yet implemented.
  Reserved for the next sprint.
- **Landing page** at `unitymbed.com` — still the main launch blocker
- **Custom domain** — point `admin.unitymbed.com` at Firebase Hosting when DNS
  is configured
- **Cloud Pro tier (Claude)** — reserved for a later phase; backend structured
  so enabling it is a config-only change

---

## Credits

Built by **UnityMbed / Voltamind AI**, with pair programming via the Claude Agent SDK.
Made with ☕ in Thailand.
