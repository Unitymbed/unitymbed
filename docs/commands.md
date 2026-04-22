# Command reference

All commands — what they do, when to use, examples. Thai + English.

---

## 🎯 Cheat sheet

| CLI | TUI (slash) | Purpose |
|-----|-------------|---------|
| `unitymbed init <name> --mcu n32g031` | — | Create project |
| `unitymbed build [debug]` | `/build` | Compile |
| `unitymbed flash` | `/flash` | Write firmware to chip |
| `unitymbed check` | `/check` | AI reads hardware registers |
| `unitymbed debug` | `/debug` | GDB interactive session |
| `unitymbed monitor` | `/monitor` | UART serial terminal |
| `unitymbed agent "<problem>"` | `/agent ...` | AI diagnostic loop |
| `unitymbed` (TUI) | | Interactive AI chat |
| — | `/ls`, `/view`, `/edit` | File ops in TUI |
| — | `/remember <fact>` | Save knowledge to project |
| — | `/new`, `/clear` | Memory management |
| `unitymbed lib add <sensor>` | `/lib add ...` | Fetch sensor driver |
| `unitymbed signup <email>` | — | Get free license |
| `unitymbed activate <KEY>` | — | Bind license to machine |
| `unitymbed usage` | — | Show quota |
| `unitymbed credits list/buy` | — | Top up Cloud AI |

---

## Project commands

### `init` — scaffold new project

Fetches the latest template from [Unitymbed/templates](https://github.com/Unitymbed/templates).

```bash
unitymbed init my-project --mcu n32g031
# --mcu options: n32g031 (64KB/8KB), n32g452 (256KB/48KB), n32g455 (512KB/144KB)
```

**สร้างโปรเจกต์ใหม่** — ดึง template จาก GitHub (Makefile, linker.ld, startup, openocd.cfg ครบ)

---

### `build [debug]` — compile firmware

```bash
unitymbed build          # -Os optimized build
unitymbed build debug    # -O0 -g3 for step-through debugging
```

Output in `build/<project>.elf` / `.bin` / `.hex`.

**Compile firmware** — ใช้ `debug` เมื่อจะใช้ GDB step ทีละบรรทัด (ไฟล์ใหญ่ขึ้นแต่ debug ได้ดี)

---

### `flash` — write firmware to chip

```bash
unitymbed flash
```

Uses the project's `openocd.cfg`. For N32G031 this is our custom TCL programmer (OpenOCD doesn't support Nations flash controller natively).

**Flash firmware ลงชิป** — ต้องต่อ CMSIS-DAP/DAPLink probe เข้ากับ SWD pins

---

### `check` — AI reads hardware state

```bash
unitymbed check
```

Flow:
1. AI reads your source files
2. Picks which MMIO registers matter
3. Halts CPU, reads each via OpenOCD
4. Shows actual vs expected table
5. If mismatches, AI explains root cause (does NOT auto-fix)

Example output:
```
┌─────────────────────────────────────────────────
│ REGISTER           ADDR       ACTUAL     EXPECT     RESULT
├─────────────────────────────────────────────────
│ RCC->APB2PCLKEN   0x40021018  0x00000008 0x00000008 ✓
│ GPIOB->PMODE      0x40010C00  0xFFFFFFFF 0xFFFF7FFF ✗ MISMATCH
└─────────────────────────────────────────────────

⚠ 1 mismatch — asking AI to analyze…

Diagnosis: GPIO mode not configured — the code's GPIO_SET_OUTPUT call
had no effect, likely because the clock isn't enabled yet.
```

**AI ตรวจ register จริง** — ดูว่า firmware ตั้งค่า hardware ถูกต้องไหม เทียบกับ source code

---

### `debug` — GDB interactive

```bash
unitymbed debug
```

Starts OpenOCD GDB server on `:3333`, launches `arm-none-eabi-gdb` connected, auto-loads ELF, breaks at `main`.

```gdb
(gdb) list               # show source
(gdb) break main.c:20    # breakpoint at line 20
(gdb) continue           # run
(gdb) next               # step over
(gdb) step               # step into
(gdb) print GPIOB->POD   # inspect register
(gdb) info registers     # CPU state
(gdb) quit
```

**Debug ทีละบรรทัด** — set breakpoint, ดูค่า variable, step through code

---

### `monitor` — UART terminal

```bash
unitymbed monitor                       # auto-detect port, 115200
unitymbed monitor --baud 9600           # custom baud
unitymbed monitor /dev/cu.usbmodem1234  # specific port
```

Exit: `Ctrl+A` then `K` then `Y`.

**เปิด UART terminal** — อ่านข้อความที่ firmware ส่งออกมา (debug printf แบบมาตรฐาน)

---

### `agent "<problem>"` — AI diagnostic loop

```bash
unitymbed agent "LED ไม่ติดที่ PB7"
```

4-step loop:
1. Build (debug mode)
2. Flash
3. Read 10 key registers
4. AI diagnoses + suggests fix (won't auto-apply)

**AI แก้ปัญหาอัตโนมัติ** — อธิบายปัญหาและบอกวิธีแก้ทันที

---

## TUI slash commands

When you run `unitymbed` (no args), you enter the TUI. Type `/` then a command:

### File ops

- `/ls [dir]` — list files in project
- `/view <path>` — show file contents with line numbers
- `/edit <path>` — open `$EDITOR` (nano by default)

### Memory

- `/remember <fact>` — persist knowledge to `.unitymbed/CONTEXT.md` (AI reads every session)
- `/new` — start fresh conversation (forget history)
- `/clear` — clear current screen (history preserved)

Example `CONTEXT.md` after a few `/remember` calls:
```markdown
## Lessons learned
- [2026-04-22] LED on this board is active-low on PB7
- [2026-04-22] External 12MHz crystal on PF0/PF1
- [2026-04-22] UART1 pins PA9 TX / PA10 RX, AF1
```

### Project actions (same as CLI)

`/build`, `/flash`, `/check`, `/monitor`, `/agent <problem>`, `/debug`, `/fix`

### Git-like cloud sync (Pro+ required)

- `/push` — sync project to Firestore
- `/pull` — pull latest from Firestore

### Help + exit

- `/help` — list commands
- `/exit` or `/quit` — save conversation + exit

---

## Library commands

The sensor library lives at [Unitymbed/sensor-library](https://github.com/Unitymbed/sensor-library).

```bash
unitymbed lib list               # show all available drivers
unitymbed lib search temp        # find drivers matching "temp"
unitymbed lib add bme280         # download into ./lib/bme280/
unitymbed lib remove bme280      # delete local copy
unitymbed lib update             # refresh registry cache
```

Current drivers: **bme280** (env), **mpu6050** (IMU), **ads1115** (ADC), **mcp2515** (CAN).

**Sensor library** — ดาวน์โหลด driver ที่เขียนไว้ให้ใช้ พร้อม header + example

---

## Account commands

### `signup <email>` — free tier

```bash
unitymbed signup me@example.com
# → email with license key → activate
```

20 Cloud AI requests/month, 10-year expiry, 1 seat.

### `activate <KEY>` — bind to machine

```bash
unitymbed activate UM-FREE-XXXX-XXXX-XXXX-XXXX
```

Stores license in `~/.unitymbed/config.yaml`. Auto-refreshes near expiry.

### `usage` — check quota

```bash
unitymbed usage
# Tier:   free
# Period: 2026-04
# Cloud:  [███░░░░░░░░░░░░░░░░░]  3/20
```

### `credits list / buy <package>`

Top up Cloud AI credits outside your monthly quota:

```bash
unitymbed credits list
unitymbed credits buy cloud_500    # ฿1,000 for +500 Cloud requests
```

Opens Stripe Checkout in your browser.

---

## Configuration file

Location: `~/.unitymbed/config.yaml`

```yaml
ai:
  provider: cloud           # ollama | cloud
  cloudAllowed: true        # false = airgap mode, forces ollama
  fallback: ollama          # if cloud unreachable
  ollamaHost: http://127.0.0.1:11434

cloud:
  enabled: true
  endpoint: https://us-central1-enterpriseunitymbed.cloudfunctions.net

license:
  key: UM-FREE-XXXX-XXXX-XXXX-XXXX
  tier: free
  email: me@example.com
  expiresAt: 2036-04-21T00:00:00.000Z
  machineId: <sha256 hash>
```

---

## See also

- [Tutorial](tutorial.md) — 10-minute BME280 sensor walkthrough
- [AI setup](ai.md) — Cloud vs Ollama in detail
- [Pricing](pricing.md) — quotas, credits, enterprise
- [Install](install.md) — toolchain setup per OS
