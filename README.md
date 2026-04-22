<div align="center">

# UnityMbed

**AI firmware assistant for ARM Cortex MCUs — built for Nations N32 series.**

Write bare-metal firmware in plain language. Build, flash, debug, and inspect hardware from a single terminal tool.

[![Version](https://img.shields.io/github/v/release/Unitymbed/homebrew-tap?label=version&color=0066ff)](https://github.com/Unitymbed/homebrew-tap/releases)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE.md)
[![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Linux%20%7C%20Windows-lightgrey)]()

[Install](#install--ติดตั้ง) · [Quick start](#quick-start--เริ่มต้น) · [Commands](docs/commands.md) · [Pricing](docs/pricing.md) · [ภาษาไทย](#ภาษาไทย-1)

</div>

---

## English

UnityMbed wraps ARM GCC, OpenOCD, and a large-language model into one workflow. It is the only tool today with a working OpenOCD flash driver for **Nations N32G031** — written from scratch after reverse-engineering the vendor's flash controller.

### Key highlights
- 🤖 **AI-native** — UnityMbed AI Cloud or local Ollama; prompts in Thai or English
- ⚡ **Bare-metal only** — no HAL, no StdPeriph. Direct MMIO writes, 196 byte blink on N32G031
- 🔧 **End-to-end** — `init → build → flash → check → debug → monitor` in one CLI
- 🔬 **AI inspection** — `/agent "LED ไม่ติด"` reads real hardware registers, diagnoses root cause
- 💾 **Project memory** — AI remembers previous conversations, code files, and your `CONTEXT.md`
- 🛒 **Sensor library** — `unitymbed lib add bme280` fetches vetted bare-metal drivers
- 💳 **Billing built-in** — Stripe subscriptions, license keys, email delivery

### Install — ติดตั้ง

**macOS / Linux** (Homebrew):
```bash
brew tap Unitymbed/tap
brew install unitymbed
```

**Windows** (PowerShell, admin):
```powershell
Invoke-WebRequest -Uri "https://github.com/Unitymbed/homebrew-tap/releases/latest/download/unitymbed-win-x64.exe" -OutFile "$env:USERPROFILE\unitymbed.exe"
```
Add `%USERPROFILE%` to PATH, then open a new terminal.

**Required toolchain** (macOS example):
```bash
brew install --cask gcc-arm-embedded   # ARM GCC
brew install open-ocd                  # flash/debug
```

See [docs/install.md](docs/install.md) for Linux/Windows equivalents.

### Quick start — เริ่มต้น

```bash
# 1. Get a free license (20 Cloud AI requests/month)
unitymbed signup your-email@example.com
unitymbed activate UM-FREE-XXXX-XXXX-XXXX-XXXX

# 2. Create a project for Nations N32G031
unitymbed init my-blink --mcu n32g031
cd my-blink

# 3. Build and flash
unitymbed build
unitymbed flash

# 4. Interactive AI session
unitymbed
# ❯ ทำให้ LED กะพริบที่ PA5 ทุก 200ms
# ❯ /build
# ❯ /flash
# ❯ /check
```

### The `/agent` workflow

When something isn't working on real hardware:

```bash
unitymbed agent "LED ไม่ติดที่ PB7"
```

UnityMbed will:
1. Rebuild in debug mode (`-O0 -g3`)
2. Flash the fresh firmware
3. Halt the CPU and read 10 key registers via OpenOCD
4. Ask the AI to diagnose root cause (clock? pin config? wiring?)
5. Suggest the exact code change — but never auto-applies it

### AI models

| Tier | Where it runs | Privacy | Cost |
|------|---------------|---------|------|
| **Unitymbed AI Local** (Ollama) | Your laptop | 🔒 Air-gapped | Free forever |
| **Unitymbed AI Cloud** | UnityMbed backend | ☁️ Prompts transit our servers | Included with paid plans |

Cloud AI is proxied through UnityMbed — there is **no bring-your-own-key**. Buy a plan, get Cloud AI. Ollama is always free.

Switch with `~/.unitymbed/config.yaml`:
```yaml
ai:
  provider: cloud          # ollama | cloud
  cloudAllowed: true       # set to false for air-gap mode (forces Ollama)
  ollamaHost: http://127.0.0.1:11434
```

Full guide: [docs/ai.md](docs/ai.md).

### Pricing

| Plan | Monthly | Cloud AI/mo | Seats |
|------|---------|-------------|-------|
| **Free** | ฿0 | 20 | 1 |
| **Start** | ฿249 | 200 | 1 |
| **Pro** | ฿499 | 1,500 | 1 |
| **Advance** | ฿1,200 | 8,000 | 3 |
| **Enterprise** | contact | unlimited | 50+ |

Ollama usage is always free (local). Pay-as-you-go credits: [docs/pricing.md](docs/pricing.md).

### Command reference

| Category | Commands |
|----------|----------|
| **Project** | `init`, `build [debug]`, `flash`, `debug`, `check`, `monitor`, `agent` |
| **AI memory** | `/remember`, `/new`, `/clear` |
| **Files (TUI)** | `/ls`, `/view`, `/edit` |
| **Library** | `lib list`, `lib search`, `lib add`, `lib remove`, `lib update` |
| **Account** | `signup`, `activate`, `usage`, `credits list`, `credits buy` |

Full docs with examples: [docs/commands.md](docs/commands.md).

### Tutorial: build a sensor in 10 minutes

[docs/tutorial.md](docs/tutorial.md) walks through setting up a BME280 temperature/humidity sensor with UART output — from blank folder to working firmware.

---

## ภาษาไทย

**UnityMbed** คือเครื่องมือสำหรับเขียน firmware ระดับ bare-metal บนชิป ARM Cortex โดยสั่งด้วยภาษาธรรมชาติ (ไทย/อังกฤษ) — เน้นชิป **Nations N32** series

### จุดเด่น
- 🤖 **AI ในตัว** — UnityMbed AI Cloud (ผ่าน backend ของเรา) หรือ Ollama ในเครื่อง
- ⚡ **Bare-metal อย่างเดียว** — ไม่ใช้ HAL/StdPeriph. เขียน register ตรงๆ ไฟล์เล็กกว่า 200 byte ก็ blink ได้
- 🔧 **End-to-end** — `init → build → flash → check → debug → monitor` ใน CLI เดียว
- 🔬 **AI ตรวจ hardware** — พิมพ์ `/agent "LED ไม่ติด"` → อ่าน register จริง → วินิจฉัยสาเหตุ
- 💾 **ความจำต่อเนื่อง** — AI จำการสนทนาเก่า, source code, CONTEXT.md
- 🛒 **Sensor library** — `unitymbed lib add bme280` ดาวน์โหลด driver พร้อมใช้

### ติดตั้ง

**Mac / Linux:**
```bash
brew tap Unitymbed/tap
brew install unitymbed
```

**Windows:** ดาวน์โหลด `.exe` จาก [releases ล่าสุด](https://github.com/Unitymbed/homebrew-tap/releases)

**Toolchain ที่ต้องมี:**
```bash
brew install --cask gcc-arm-embedded   # ARM GCC
brew install open-ocd                  # flash/debug
```

### เริ่มใช้งาน

```bash
# 1. สมัครฟรี (Cloud AI 20 req/เดือน)
unitymbed signup your-email@example.com
# → ตรวจ inbox รับ license key
unitymbed activate UM-FREE-XXXX-XXXX-XXXX-XXXX

# 2. สร้างโปรเจกต์
unitymbed init my-blink --mcu n32g031
cd my-blink

# 3. Build + flash
unitymbed build
unitymbed flash

# 4. เปิด TUI สั่งงาน AI
unitymbed
# ❯ ทำให้ LED กะพริบที่ PA5 ทุก 200ms
# ❯ /build
# ❯ /flash
```

### ราคา

| แพ็คเกจ | /เดือน | Cloud AI/เดือน | Seats |
|---------|-------|----------------|-------|
| **Freemium** | ฿0 | 20 | 1 |
| **Start** | ฿249 | 200 | 1 |
| **Pro** | ฿499 | 1,500 | 1 |
| **Advance** | ฿1,200 | 8,000 | 3 |
| **Enterprise** | ติดต่อ | ไม่จำกัด | 50+ |

Ollama (ในเครื่อง) **ใช้ฟรีตลอด** — ซื้อเฉพาะ cloud AI

### เอกสารเพิ่มเติม

- [คำสั่งทั้งหมด (commands)](docs/commands.md)
- [ราคา (pricing)](docs/pricing.md)
- [ตั้งค่า AI](docs/ai.md)
- [ติดตั้ง (install)](docs/install.md)
- [สอนใช้งาน (tutorial)](docs/tutorial.md)

---

<div align="center">

**Website:** [unitymbed.com](https://unitymbed.com)  ·  **Support:** support@unitymbed.com  ·  **License:** [MIT](LICENSE.md)

Made with ☕ in Thailand

</div>
