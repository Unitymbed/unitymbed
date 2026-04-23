# UnityMbed — Revision History

AI firmware assistant for ARM Cortex MCUs (Nations N32 series).
Full-stack: CLI + Firebase backend + admin web app + managed knowledge base.

> This document chronicles every public release from `v0.1.0` to `v0.3.2`.

---

## Project scale snapshot (v0.3.2, 2026-04-23)

| Metric | Value |
|---|---|
| **Public releases** | 21 (`v0.1.0` → `v0.3.2`) |
| **Total source LOC** | 5,593 (TypeScript/TSX) |
| &nbsp;&nbsp;`apps/cli/src` | 2,972 lines (46 files) |
| &nbsp;&nbsp;`apps/functions/src` | 1,554 lines (18 files) |
| &nbsp;&nbsp;`apps/functions/scripts` | 334 lines (4 files) |
| &nbsp;&nbsp;`apps/admin/src` | 733 lines (10 files) |
| **Cloud Functions deployed** | 19 |
| **Firebase Hosting sites** | 1 (`unitymbed-admin.web.app`) |
| **Knowledge base chunks** | 2,341 (N32G031: DS + UM + Nations SDK) |
| **Supported platforms** | macOS arm64/x64, Linux x64, Windows x64 |
| **Distribution** | Homebrew tap + GitHub releases |
| **Estimated build time** | ~45–60 hours over ~3 days of focused work |

---

## Architecture (current state)

```
┌──────────────────────┐      ┌──────────────────────────────────┐
│ Customer CLI (brew)  │      │  Admin Web GUI                   │
│ unitymbed v0.3.2     │      │  https://unitymbed-admin.web.app │
│  • TUI chat          │      │  • Google login + allowlist      │
│  • init/build/flash  │      │  • KB CRUD + upload              │
│  • /agent /check     │      │  • Test query (RAG debug)        │
│  • license+quota     │      │  • Admin management              │
└──────────┬───────────┘      └──────────────┬───────────────────┘
           │                                 │
           │  SSE                            │  signed URLs + Bearer JWT
           ▼                                 ▼
┌──────────────────────────────────────────────────────────────────┐
│ Firebase (enterpriseunitymbed)                                   │
│ ┌──────────────┐ ┌─────────────┐ ┌──────────────┐ ┌───────────┐ │
│ │ aiProxy      │ │ kbProcess   │ │ adminList    │ │ kbList    │ │
│ │  +RAG inline │ │ (Storage    │ │ adminAdd     │ │ kbDelete  │ │
│ │  Gemini-side │ │  trigger)   │ │ adminRemove  │ │ kbTestQ.  │ │
│ │  identity    │ │             │ │              │ │ kbUpload  │ │
│ └──────────────┘ └─────────────┘ └──────────────┘ └───────────┘ │
│ ┌──────────────────────────────────────────────────────────────┐ │
│ │ Firestore                                                    │ │
│ │  licenses/ • customers/ • admins/ • kb_docs/ • kb_chunks/    │ │
│ │              (vector-indexed 768-dim COSINE)                 │ │
│ └──────────────────────────────────────────────────────────────┘ │
│ ┌───────────────┐ ┌──────────────┐ ┌──────────────────────────┐ │
│ │ Stripe        │ │ Resend       │ │ Secret Manager           │ │
│ │ webhook       │ │ email        │ │  GEMINI / STRIPE / RESEND│ │
│ └───────────────┘ └──────────────┘ └──────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

---

## Cloud Functions inventory (19)

| Function | Purpose |
|---|---|
| `onUserCreate` | Firebase Auth trigger — provision customer doc |
| `activateLicense` | Bind a license key to a machine ID |
| `checkLicense` | Periodic refresh from CLI |
| `checkQuota` | Pre-flight quota check (legacy; retained) |
| `getUsage` | Usage summary for CLI `/usage` |
| `createFreeLicense` | `unitymbed signup` backend |
| `stripeWebhook` | Stripe subscription lifecycle + renewal + credit purchase emails |
| `buyCredits` | Pay-as-you-go credit Checkout |
| `syncUpload` / `syncDownload` | Project cloud backup |
| `aiProxy` | **Core AI path** — SSE, license+quota+identity, RAG injection, Gemini proxy |
| `adminList` / `adminAdd` / `adminRemove` | Admin allowlist management |
| `kbList` / `kbDelete` / `kbTestQuery` / `kbUploadUrl` | KB admin API |
| `kbProcess` | **Storage trigger** (asia-southeast1) — parse PDF → chunk → embed → write |

---

# Version-by-version history

All dates in UTC. Every release is tagged on `Unitymbed/homebrew-tap` with pre-built binaries for macOS (arm64 + x64), Linux x64, and Windows x64.

---

## `v0.1.0` — 2026-04-21  *Initial public release*

First public build. Distributed via Homebrew tap `Unitymbed/tap`.

Baseline capabilities:
- Cross-platform binary (`bun --compile`) for macOS / Linux / Windows
- Ink + React TUI with streaming AI responses
- Three AI backends: Ollama (local), Google Gemini (cloud), Anthropic Claude (cloud pro) — all BYOK at this stage
- Project scaffolding: `unitymbed init <name> --mcu n32g031`
- ARM GCC build pipeline, initial OpenOCD config iterations
- YAML config at `~/.unitymbed/config.yaml`

Install: `brew tap Unitymbed/tap && brew install unitymbed`

---

## `v0.1.1` — 2026-04-21  *Brand labels*

- Rebrand AI provider display name to **Unitymbed AI Cloud / Local** in the status bar
- Begin insulating users from the underlying model name

---

## `v0.1.2` — 2026-04-21  *Tier naming*

Locks the three-tier brand that persists into v0.3.x:
- **Claude → Unitymbed AI Cloud Pro**
- **Gemini → Unitymbed AI Cloud**
- **Ollama → Unitymbed AI Local**

---

## `v0.1.3` — 2026-04-21  *Usage quota + `unitymbed usage`*

First billing scaffolding.
- Per-tier monthly quota tracking (Free / Start / Pro / Advance)
- New command: `unitymbed usage` shows remaining cloud / cloudPro requests this month
- Pay-as-you-go credit field prepared in Firestore (not yet purchasable)

---

## `v0.1.4` — 2026-04-21  *Credits + Stripe Checkout*

- `unitymbed credits list` / `unitymbed credits buy <pack>` via Stripe Checkout
- Two credit packs: `cloud_100`, `cloudPro_100`
- Multi-key config support (`keys.gemini` + `keys.claude` in the same YAML file)

---

## `v0.1.5` — 2026-04-21  *License renewal + auto-refresh*

Closes the subscription lifecycle:
- Auto-refresh license on disk when local expiry nears (transparent to the user)
- Stripe `invoice.paid` webhook extends expiry by 31 days
- `customer.subscription.deleted` marks the license as `willNotRenew`

---

## `v0.1.6` — 2026-04-21  *Quota rebalance; Cloud Pro off*

- Claude tier (Cloud Pro) temporarily disabled system-wide — reduces support surface while core paths stabilize
- Increased Cloud quota caps for paid tiers:
  - Start 200 → Pro 1,500 → Advance 8,000 requests / month

---

## `v0.1.7` — 2026-04-21  *Single-provider cloud*

- All tiers route to Gemini (`Unitymbed AI Cloud`)
- Explicit messaging so users stop expecting Claude

---

## `v0.1.8` — 2026-04-21  *Free signup flow*

- New command: `unitymbed signup <email>` → zero-payment license (20 Cloud requests / month)
- Backend function `createFreeLicense` (one-per-email guard)
- Resend email with license key
- Removes the "you need a credit card to try it" barrier

---

## `v0.1.9` — 2026-04-21  *N32G031 flash works* 🎉

Landmark release — **first public OpenOCD flash driver for Nations N32G031**.
- `unitymbed init` now fetches templates from `Unitymbed/templates` on GitHub (fixes broken `cp` path in compiled binary)
- Custom TCL flash programmer writes directly to the N32G031 flash controller at `0x40022000` (unlock keys `0x45670123` / `0xCDEF89AB`), bypassing OpenOCD's `stm32f1x` driver which fails because Nations places DBGMCU at `0x1FFFF508` (not the STM32 convention at `0x40015800`)
- End-to-end `init → build → flash` verified on real hardware — LED blink on PA8

---

## `v0.2.0` — 2026-04-21  *Production Ready*

Promotion milestone. Everything stabilized and shippable.
- N32G031 flash end-to-end re-verified on real hardware (LED blink)
- AI system prompt now embeds a **full N32G031 register reference card** (memory map, clock tree, GPIO registers, USART1, Flash controller) so responses are grounded even without RAG
- New datasheet repo: https://github.com/Unitymbed/datasheets
- Template fix: correct APB2 clock bits for GPIO (N32G031 uses `APB2PCLKEN`, **not** AHB like STM32F1)
- Stripe Live pipeline end-to-end verified with a real ฿249 transaction

---

## `v0.2.1` — 2026-04-21  *Memory & Context*

Conversations and project awareness, per-project.
- Conversation history persisted to `.unitymbed/conversations/` — auto-loads last session when reopening the TUI
- Automatic project context injection on every prompt: `src/*`, `unitymbed.json`, and `CONTEXT.md`
- New `/new` command to start a fresh conversation

---

## `v0.2.2` — 2026-04-22  *Hardware Inspection*

First tools that actually reach into the running chip.
- `/check` — AI picks which MMIO registers matter for the current problem, OpenOCD halts + reads them, output is shown as a table
- `/check` mismatches — AI analyses why expected ≠ observed and **suggests** a fix (never auto-edits code — user feedback)
- `/monitor` — auto-detects the CMSIS-DAP virtual COM port and opens a 115,200-baud serial terminal

---

## `v0.2.3` — 2026-04-22  *Auto-update notifier*

Small but important release hygiene.
- Background check against GitHub releases once every 24h
- Non-intrusive hint in TUI header and after CLI commands when a newer version is available
- New: `unitymbed --version`

---

## `v0.2.4` — 2026-04-22  *Agent + Thai Input*

Two unrelated wins landed together.
- `unitymbed build debug` compiles with `-O0 -g3` so GDB single-step works
- `unitymbed agent "<problem>"` — 4-step autonomous loop: build (debug) → flash → register snapshot → AI diagnosis
- `/remember <fact>` — persists a lesson to the project's `CONTEXT.md`
- Thai input in Ink TUI: show preview above the input box when Thai characters are detected, hide cursor to avoid combining-mark column misalignment

---

## `v0.2.5` — 2026-04-22  *`/agent` inside the TUI*

- The agent loop runs inline inside the TUI instead of spawning a raw console
- Progress renders as typed tool-result messages
- Graceful error surfaces when hardware is disconnected

---

## `v0.2.6` — 2026-04-22  *File Operations in TUI*

Inspired by user request "ขอหน้าต่างให้ edit file และเรียกดูไฟล์ใน TUI ได้ไหม":
- `/ls [dir]` — lists project files
- `/view <path>` — shows file content with line numbers
- `/edit <path>` — opens `$EDITOR` (nano by default) on the file

---

## `v0.2.7` — 2026-04-22  *Branded Identity*

- AI always introduces itself as **"UnityMbed AI"**
- Never mentions Google / Gemini / Anthropic / Claude / Ollama / OpenAI / ChatGPT / any other AI vendor or model name
- Prevents branding leaks even when the backing model goes off-script
- Identity rule embedded in the CLI's system prompt (later moved server-side in v0.3.0)

---

## `v0.3.0` — 2026-04-22  *Unitymbed AI Cloud (no BYOK)*

Major architectural shift. Cloud AI is proxied through UnityMbed's backend only — no more user-configured API keys.

**Breaking changes:**
- `ai.provider` accepts only `"ollama"` or `"cloud"`
- `ai.apiKey` and `ai.keys.*` stripped from the config schema
- Legacy `gemini` / `claude` provider values auto-migrate to `"cloud"` on first launch; any on-disk keys are scrubbed

**Backend:**
- New `aiProxy` Cloud Function — validates license + machine + quota via Firestore, calls Gemini with our `GEMINI_API_KEY` (Secret Manager), streams SSE back
- **Identity rule hardened server-side** — prepended to every request before any client system prompt, so direct callers can't bypass it
- Quota deducted only on successful generation

**CLI:**
- Dropped `@anthropic-ai/sdk` and `@google/generative-ai` dependencies
- Removed `claude.ts`, `gemini.ts`, `quotaWrapper.ts`
- New `UnityMbedCloudClient` that speaks to `aiProxy` over SSE
- `config.migrate()` runs on every `loadConfig()` and persists cleaned config back to disk

**Docs:** `Unitymbed/unitymbed` README + `docs/ai.md` + `docs/commands.md` + `docs/pricing.md` rewritten to drop BYOK and mark Cloud Pro (Claude) as reserved for a later phase.

---

## `v0.3.1` — 2026-04-23  *RAG Grounded Answers*

Every Cloud query is now automatically grounded in a curated knowledge base. Zero user action required.

**Backend:**
- New `apps/functions/src/ai/rag.ts` — Gemini `gemini-embedding-001` embedding with 768-dim Matryoshka truncation, Firestore native vector search (`findNearest`, COSINE distance), graceful fail-open on any retrieval error (RAG must never break chat)
- `aiProxy` extended with `mcu` / `useKB` / `topK` body fields. Retrieves top-5 chunks for the last user message, prepends a `GROUNDED REFERENCES` block after the identity rule and firmware reference card. Emits `data: {ragHits: N}` SSE frame before the first token.

**Admin ingestion pipeline** (runs locally, `apps/functions/scripts/`):
- `lib/pdf.ts` — server-side PDF text extraction via `pdfjs-dist` legacy build (no worker)
- `lib/chunk.ts` — prose chunker (~1,400 chars with paragraph-aware breaks) + boundary-aware code chunker (C/H function definitions)
- `lib/embed.ts` — concurrency-6 batched embedder with exponential backoff on 429/5xx
- `ingest.ts` — CLI: `bun scripts/ingest.ts <path> --mcu n32g031 --type user_manual`

**CLI:**
- `UnityMbedCloudClient` reads `mcu` from the nearest `unitymbed.json` and sends it with every chat request
- Renders **"📚 grounded with N refs"** when retrieval fires

**KB seed at launch:**
- N32G031 Datasheet (86 pages) → **148 chunks**
- N32G031 User Manual (550 pages) → **958 chunks**
- Nations N32G031 SDK firmware (.c/.h) → **1,235 chunks**
- **Total: 2,341 chunks**, all scoped by MCU tag

**Firestore vector index:** `kb_chunks` collection group, composite `(mcu ASC, embedding VECTOR 768 flat)`, COSINE distance.

---

## `v0.3.2` — 2026-04-23  *Hotfix: Cloud default for fresh installs*

Discovered when a second user (`por@voltamine-ai.com`) tried v0.3.1:
```
unitymbed signup por@voltamine-ai.com       # OK
unitymbed activate UM-FREE-…                # OK
unitymbed                                    # → "AI error: AI provider 'ollama' is not available"
```

Root cause: `DEFAULT_CONFIG.ai.provider` was still `"ollama"` (legacy). After activation the config didn't switch to `"cloud"`, and the user had no Ollama installed.

**Fixes:**
- `DEFAULT_CONFIG.ai.provider` flipped from `ollama` → `cloud`
- `unitymbed activate` auto-switches an `ollama`-defaulted config to `cloud` and normalizes the model field
- `createProvider()` router prefers Cloud whenever a license is present (belt-and-suspenders for users upgrading with a stale on-disk config)
- `Header` shows `license.email` instead of `anonymous` once activated

Airgap mode is unchanged — users who explicitly want local Ollama still set `provider: ollama` + `cloudAllowed: false`.

---

# Beyond the CLI — companion deliverables this cycle

## Admin Web GUI  — `unitymbed-admin.web.app`
Shipped alongside v0.3.1. Full React + Vite + TailwindCSS SPA.

- **Auth:** Firebase Google sign-in; allowlist stored in Firestore `admins/{email}`. Non-admins see "Not authorized".
- **Pages:**
  - **Documents** — list/filter by MCU, show chunk counts, delete (cascades to all chunks in batched writes)
  - **Upload** — drag-drop PDF, reserves a `kb_docs` doc ID, receives a signed Storage URL, uploads directly to bucket; `kbProcess` Storage trigger handles the rest
  - **Test Query** — embed a test question, see top-K hits with source file, page, and raw text (debug tool for KB coverage)
  - **Admins** — add/remove admin emails (cannot remove yourself)
- **8 new Cloud Functions** back the GUI: `adminList` / `adminAdd` / `adminRemove` / `kbList` / `kbDelete` / `kbTestQuery` / `kbUploadUrl` / `kbProcess` (Storage trigger, `asia-southeast1`)

## Custom domain authorization
Added `unitymbed-admin.web.app` + `unitymbed-admin.firebaseapp.com` to the Firebase Auth authorized-domains list via Identity Toolkit REST API.

## Documentation repo — `Unitymbed/unitymbed`
- Rewritten for Phase-A architecture (no BYOK, two AI tiers: Local + Cloud)
- All mentions of `keys.gemini` / `keys.claude` and Cloud Pro (Claude) removed
- `docs/ai.md` completely rewritten with the new model

---

# Complexity highlights

| Subsystem | ⭐ | Notes |
|---|---|---|
| **Custom OpenOCD N32G031 flash driver** | ⭐⭐⭐⭐⭐ | Reverse-engineered Nations flash controller from vendor SDK; custom TCL unlock sequence; first-of-its-kind in the public ecosystem |
| **Stripe → license → email pipeline** | ⭐⭐⭐⭐ | 4 webhook event types × 3 tiers × email delivery × credit metadata branching × signature verification |
| **RAG retrieval inside the same SSE request** | ⭐⭐⭐⭐ | Embed query → Firestore vector `findNearest` → context assembly → fail-open on error, all *inside* the generation round-trip |
| **Admin GUI with Firebase Auth + allowlist** | ⭐⭐⭐ | ID-token verification → Firestore admin check → CORS allowlist per endpoint → signed-URL upload path → Storage trigger chain |
| **Ingestion pipeline** | ⭐⭐⭐ | pdfjs-dist legacy build (no worker), regex-based function-boundary code chunker, concurrent embedding with backoff |
| **Config migration (BYOK → managed)** | ⭐⭐⭐ | Auto-strips on-disk keys, remaps legacy provider names, persists only on actual change |
| **Thai input rendering in Ink TUI** | ⭐⭐ | Thai combining marks misalign terminal columns — preview-above-input + cursor-hidden fixes it |
| **Cross-platform binary build** | ⭐⭐ | `bun build --compile --target=bun-{mac-arm64,mac-x64,linux-x64,windows-x64}` |

---

# External integrations

| Service | Usage |
|---|---|
| **Stripe (Live)** | Subscriptions — Start ฿249, Pro ฿499, Advance ฿1,200 — plus credit-pack Checkouts |
| **Resend** | Transactional email (license, renewal, credit receipt); currently `onboarding@resend.dev`, pending DNS verification for `noreply@unitymbed.com` |
| **Firebase Auth** | Admin Google sign-in + ID-token verification on backend |
| **Firestore** | 5 top-level collections — `licenses`, `customers`, `admins`, `kb_docs`, `kb_chunks`; vector index on `kb_chunks.embedding` |
| **Firebase Storage** | `kb/{docId}/` admin uploads (signed-URL write, Functions-only read) |
| **GCP Secret Manager** | `GEMINI_API_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY` |
| **Gemini API** | `gemini-2.5-flash` for generation, `gemini-embedding-001` for 768-dim embeddings |
| **GCP Service Accounts** | `kb-ingest@…` for local admin ingestion (Firestore write + Firebase Admin role) |
| **Homebrew tap** | `Unitymbed/homebrew-tap` — formula auto-updated each release |
| **GitHub Releases** | 21 tagged releases with pre-built binaries (macOS arm64/x64, Linux x64, Windows x64) |

---

# Deployed URLs

| Resource | URL |
|---|---|
| AI Cloud proxy | `https://us-central1-enterpriseunitymbed.cloudfunctions.net/aiProxy` |
| Admin web app | `https://unitymbed-admin.web.app` |
| Main docs repo | `https://github.com/Unitymbed/unitymbed` |
| Homebrew tap | `https://github.com/Unitymbed/homebrew-tap` |
| Templates | `https://github.com/Unitymbed/templates` |
| Sensor library | `https://github.com/Unitymbed/sensor-library` |
| Datasheets | `https://github.com/Unitymbed/datasheets` |

---

# Open items

- **Gemini key rotation** — the key used in Secret Manager was shared via chat during setup; recommended to rotate at https://aistudio.google.com/app/apikey → update Secret Manager → redeploy `aiProxy` / `kbProcess`
- **Resend DNS verification** — switch sender from `onboarding@resend.dev` to `noreply@unitymbed.com`
- **Model routing by tier** — currently all tiers get `gemini-2.5-flash`; design pending (tier-based, user-toggled, or smart auto-select)
- **Cloud Pro tier (Claude)** — reserved for a later phase; backend structured so enabling it is a config-only change (`CLOUD_PRO_ENABLED = true` + Anthropic secret)
- **Additional MCU KBs** — N32G452 and N32G455 datasheets / manuals to ingest (Admin Web GUI is ready to accept them)
- **Custom domain** — point `admin.unitymbed.com` at Firebase Hosting once DNS is set
- **Landing page** at `unitymbed.com` — critical blocker for public launch

---

## Credits

Built by **UnityMbed / Voltamind AI**, with pair programming via the Claude Agent SDK.
Made with ☕ in Thailand.
