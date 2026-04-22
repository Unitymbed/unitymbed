# AI setup — Cloud & Local

UnityMbed supports 3 AI backends. Pick based on privacy / speed / cost.

---

## Summary / สรุป

| Backend | Speed | Quality | Privacy | Cost |
|---------|-------|---------|---------|------|
| **Ollama** (local) | ⚡⚡ | ⭐⭐⭐ | 🔒 100% local | Free |
| **Gemini** (cloud) | ⚡⚡⚡ | ⭐⭐⭐⭐ | ☁️ Google | Monthly plan or BYOK |
| **Claude** (cloud pro) | ⚡⚡ | ⭐⭐⭐⭐⭐ | ☁️ Anthropic | Premium tier or BYOK |

---

## Option 1: Ollama — local, free, private

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
  cloudAllowed: false
```

### Pros & cons

✅ Free forever, no internet, no account  
✅ Code never leaves your machine  
❌ Slower (10-30s/response on M1 Mac)  
❌ Smaller models = lower accuracy vs cloud  
❌ Needs 8GB+ RAM

---

## Option 2: Gemini — cloud, best balance

### Get API key

1. Go to https://aistudio.google.com/app/apikey
2. Click **Create API key**
3. Copy `AIza...` string

### Configure

```yaml
ai:
  provider: gemini
  model: gemini-2.5-flash        # or gemini-2.5-pro for harder tasks
  keys:
    gemini: AIza...               # your key
  cloudAllowed: true
  fallback: ollama                # fallback if cloud fails
```

### Google's free tier (BYOK)

If you use your own key:
- **gemini-2.5-flash**: 500 req/day free
- **gemini-2.5-pro**: 50 req/day free

That's 15,000 req/month free — more than our Advance plan — but requires you to manage the Google account yourself.

### Pros & cons

✅ Fast (2-5s/response)  
✅ Best for structured code generation  
✅ Free tier via AI Studio (if BYOK)  
❌ Requires internet  
❌ Prompts visible to Google (not stored, but in transit)

---

## Option 3: Claude — premium reasoning

**Note:** Cloud Pro (Claude) is temporarily disabled system-wide. Coming back soon.

### Get API key

1. Go to https://console.anthropic.com/settings/keys
2. Add billing credit ($5 min)
3. Create key → `sk-ant-...`

### Configure

```yaml
ai:
  provider: claude
  model: claude-sonnet-4-6
  keys:
    claude: sk-ant-...
  cloudAllowed: true
  fallback: ollama
```

### Pros & cons

✅ Best at multi-step debugging  
✅ Long-context understanding of codebase  
❌ Most expensive (~$3/M input tokens, $15/M output)  
❌ Slower than Gemini

---

## Hybrid setup (recommended for power users)

Mix and match — use Ollama for quick edits, Gemini for generation, Claude for hard bugs:

```yaml
ai:
  provider: gemini                # default for TUI prompts
  model: gemini-2.5-flash
  keys:
    gemini: AIza...
    claude: sk-ant-...
  cloudAllowed: true
  fallback: ollama                # auto-fallback if Gemini fails
```

Switch provider on the fly by editing `~/.unitymbed/config.yaml` and restarting `unitymbed`.

---

## Privacy toggle — airgap mode

If you never want code to leave your machine:

```yaml
ai:
  provider: ollama
  cloudAllowed: false       # hard-disables all cloud paths
```

With `cloudAllowed: false`:
- Even if `provider: gemini`, calls are blocked
- `/push`, `/pull` still work only if you explicitly set `cloud.enabled: true`
- License check happens once per week via Firestore — you can skip it by setting `cloud.enabled: false` (but renewals won't auto-refresh)

---

## Current system prompt

UnityMbed embeds a **bare-metal N32G031 reference card** in every prompt. AI knows:
- Register map (RCC, GPIO, USART, Flash controller)
- Correct clock bits (APB2PCLKEN for GPIO, not AHB)
- Common pins (UART1 = PA9/PA10, etc.)
- Flash programming sequence

This is why prompts like "blink LED at PB7" produce correct code on the first try, regardless of AI model.

Full datasheet reference: [Unitymbed/datasheets](https://github.com/Unitymbed/datasheets)
