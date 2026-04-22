# AI setup — Cloud & Local

UnityMbed ships with two AI backends. Pick based on privacy, speed, and budget.

---

## Summary / สรุป

| Backend | Speed | Quality | Privacy | Cost |
|---------|-------|---------|---------|------|
| **Unitymbed AI Local** (Ollama) | ⚡⚡ | ⭐⭐⭐ | 🔒 100% on your laptop | Free forever |
| **Unitymbed AI Cloud** | ⚡⚡⚡ | ⭐⭐⭐⭐ | ☁️ Through UnityMbed backend | Included with paid plans |

> **No bring-your-own-key.** Cloud AI traffic is proxied through UnityMbed —
> we hold the provider key, enforce quota, and handle billing. If you want
> air-gapped operation, use Ollama.

---

## Option 1: Unitymbed AI Local (Ollama) — free, private

Runs the model on your own machine. No license required. Never contacts the network.

### Install

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows: download installer from https://ollama.com/download
```

### Pull a model

```bash
ollama serve &                    # start daemon
ollama pull qwen2.5-coder:7b      # recommended for embedded code
# or: codellama:7b, codeqwen, deepseek-coder
```

### Configure UnityMbed

`~/.unitymbed/config.yaml`:
```yaml
ai:
  provider: ollama
  model: qwen2.5-coder:7b
  ollamaHost: http://127.0.0.1:11434
  cloudAllowed: false          # hard-disables all cloud paths
```

### Pros & cons

✅ Free forever, no internet, no account
✅ Code never leaves your machine
❌ Slower (10–30s / response on M1 Mac)
❌ Smaller models = lower accuracy vs Cloud
❌ Needs 8 GB+ RAM

---

## Option 2: Unitymbed AI Cloud — best quality, included with paid plans

Traffic goes to UnityMbed's `aiProxy` endpoint which validates your license, enforces quota, and forwards to our backing model using our key.

### Requirements

- An active UnityMbed license (any paid tier, or a Freemium signup which
  includes **20 Cloud AI requests / month**).
- Your license key activated on this machine:
  ```bash
  unitymbed activate UM-XXXX-XXXX-XXXX-XXXX-XXXX
  ```

### Configure

```yaml
ai:
  provider: cloud
  cloudAllowed: true
  fallback: ollama        # optional: fall back to local if Cloud is unreachable
```

That's it. **Do not** put an API key in this file — Cloud AI is not BYOK.

### Quota

Check your monthly usage any time:
```bash
unitymbed usage
```

Need more? Upgrade your plan or buy pay-as-you-go credits:
```bash
unitymbed credits buy cloud_100
```

See [pricing.md](pricing.md) for current caps.

### Pros & cons

✅ Fastest (2–5 s / response)
✅ Highest quality on embedded-specific tasks
✅ No API key or billing setup — one purchase covers everything
❌ Requires internet
❌ Prompts transit UnityMbed servers (not stored long-term; see privacy note below)

---

## Hybrid setup (recommended)

Default to Cloud for hard problems, but fall back to Ollama if offline:

```yaml
ai:
  provider: cloud
  cloudAllowed: true
  fallback: ollama
ollamaHost: http://127.0.0.1:11434
```

Switch manually any time by editing `~/.unitymbed/config.yaml` and restarting `unitymbed`.

---

## Airgap mode

Never want anything to leave your machine?

```yaml
ai:
  provider: ollama
  cloudAllowed: false
```

With `cloudAllowed: false`:
- Any `provider: cloud` call is blocked before hitting the network.
- `unitymbed push`, `unitymbed pull` still work only if `cloud.enabled: true`.
- License check still runs weekly (to refresh expiry). Disable with `cloud.enabled: false` if you want a fully offline box — but auto-renewals won't propagate.

---

## Privacy — what leaves your machine when you use Cloud AI

When `provider: cloud`, each request sends to UnityMbed:

- Your license key + machine ID (for quota)
- The prompt you typed
- The current project context (CONTEXT.md, open files snippets, conversation history for that project)

UnityMbed forwards the prompt to the backing model and streams the response back. We do not retain prompt bodies after the request completes. License/quota metadata is retained for billing.

If that's not acceptable for your codebase → use Ollama.

---

## Embedded system prompt

Every request (Local or Cloud) carries a built-in bare-metal N32G031 reference card so the model knows:

- Register map (RCC, GPIO, USART, Flash controller)
- Correct clock bits (APB2PCLKEN for GPIO — **not** AHBPCLKEN)
- Default pin mappings (UART1 = PA9/PA10, etc.)
- Flash programming sequence

This is why prompts like "blink LED at PB7" produce correct code on the first try.

Full register reference: [Unitymbed/datasheets](https://github.com/Unitymbed/datasheets)

---

## Coming later

A premium **Unitymbed AI Cloud Pro** tier (built on a stronger reasoning model for multi-step debugging) is on the roadmap but disabled today. When it ships you won't need to change configuration — just upgrade your plan.
