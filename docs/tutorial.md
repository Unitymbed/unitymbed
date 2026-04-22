# Tutorial — 10-minute blinky + UART

Build your first working N32G031 firmware from scratch. No embedded experience needed.

---

## What you'll learn / สิ่งที่จะได้เรียน

By the end:
- ✅ LED blinking on PB7
- ✅ UART1 sends "hello #N" each blink
- ✅ You can edit code with AI and re-flash in 1 command

---

## Prerequisites / เตรียม

- ✅ [Installed UnityMbed](install.md) — `unitymbed --version` works
- ✅ [Activated a license](install.md#test) — free signup is fine
- ✅ N32G031 dev board connected via CMSIS-DAP probe (SWD pins)
- ✅ LED wired to PB7 (or use your board's onboard LED, confirm pin)

---

## Step 1: Create project / สร้างโปรเจกต์

```bash
mkdir ~/blinky && cd ~/blinky
unitymbed init hello --mcu n32g031
cd hello
```

You'll see:
```
→ Fetching template n32g031 from GitHub…
✓ Created /Users/you/blinky/hello
  MCU: n32g031 (cortex-m0, 64KB flash, 8KB RAM)
  Next: cd hello && unitymbed build
```

File tree:
```
hello/
├── Makefile
├── README.md
├── include/
│   └── n32g031.h           # MMIO struct defs + helper macros
├── linker.ld               # flash + RAM layout
├── openocd.cfg             # custom TCL flash programmer
├── src/
│   └── main.c              # default blink on PA8
├── startup_n32g031.s       # Cortex-M0 vector table + reset handler
└── unitymbed.json          # project manifest
```

---

## Step 2: Write context for AI / เขียนคำอธิบายให้ AI

Tell the AI about your board so every prompt gets grounded in reality:

```bash
cat > .unitymbed/CONTEXT.md <<'EOF'
# My N32G031 Dev Board

- MCU: N32G031K8U7, 64KB flash, 8KB SRAM, 48 MHz max, default HSI 8MHz
- Debug probe: CMSIS-DAP-JYD via SWD (SWCLK = PA14, SWDIO = PA13)
- LED: PB7 (active-high, external 220Ω series to 3V3)
- UART for logging: USART1 @ 115200 — PA9 TX, PA10 RX (AF1)

## Known quirks
- GPIO clocks on N32G031 live in RCC->APB2PCLKEN (NOT AHBPCLKEN)
  - GPIOA = bit 2, GPIOB = bit 3, GPIOC = bit 4
- SysTick default 1ms tick not set up yet
EOF
```

This file is read **every time** AI runs — it never forgets.

---

## Step 3: Prompt the AI / สั่ง AI

```bash
unitymbed
```

You're in the TUI. Type (Thai or English, both work):

```
❯ แก้ main.c ให้ blink LED PB7 ทุก 300ms และส่ง "hello #<count>\r\n" ผ่าน UART1 115200
```

AI reads:
- Your `CONTEXT.md` (knows LED on PB7, UART1 on PA9/PA10)
- Current `src/main.c`, `include/n32g031.h`
- Embedded N32G031 register reference

AI responds with updated `src/main.c` that sets up RCC, GPIO, USART1, and blinks. It auto-writes the file.

---

## Step 4: Build / Compile

In the TUI:
```
❯ /build
```

Or from shell:
```bash
unitymbed build
```

Expected output:
```
arm-none-eabi-gcc ... -o build/src/main.o
arm-none-eabi-gcc ... -o build/hello.elf
arm-none-eabi-objcopy -O binary build/hello.elf build/hello.bin
   text   data    bss    dec    hex
    342      0      0    342    156  build/hello.elf
```

---

## Step 5: Flash to chip / เขียนลงชิป

```
❯ /flash
```

Watch the output:
```
N32: flash mass-erased
N32: programming 342 bytes @ 0x08000000
N32: programmed 342 bytes
[n32g031.cpu] external reset detected
```

**LED on PB7 should be blinking now.** 🎉

---

## Step 6: See UART output / ดูข้อความ UART

In a **new terminal** (keep TUI open):

```bash
unitymbed monitor
```

You'll see:
```
→ Opening /dev/cu.usbmodem0001A00000021 @ 115200 baud
hello #0
hello #1
hello #2
hello #3
...
```

Exit: `Ctrl+A` then `K` then `Y`.

---

## Step 7: Iterate with AI / แก้โปรแกรมต่อ

Back in the TUI:

```
❯ เปลี่ยน delay เป็น 100ms
```

AI updates the delay constant. Then:
```
❯ /build
❯ /flash
```

LED blinks faster. No re-wiring, no IDE menus.

---

## Step 8: Add a sensor (BME280) / เพิ่ม sensor

```
❯ /lib add bme280
```

Downloads `lib/bme280/{bme280.c, bme280.h, README.md}` from Unitymbed/sensor-library.

Then:
```
❯ ต่อ BME280 บน I2C1 (PA11 SDA, PA12 SCL) อ่านอุณหภูมิและความชื้นทุก 1s ส่ง UART เป็น JSON
```

AI generates code that calls BME280 driver, sets up I2C1, formats JSON output.

---

## Step 9: Debug if something breaks / ถ้ามีปัญหา

If LED doesn't blink:

```
❯ /agent LED ไม่ติดที่ PB7
```

UnityMbed runs:
1. Debug build (-O0 -g3)
2. Flash
3. Read 10 registers (RCC, GPIO*)
4. AI diagnoses based on actual hardware state

Example diagnosis:
```
Diagnosis: GPIOB clock not enabled
Root cause: RCC->APB2PCLKEN value = 0x00000000 (expected 0x00000008)
Suggested fix: Add `RCC->APB2PCLKEN |= (1 << 3);` at the start of main()
Why: Nations N32G031 uses APB2 clock for GPIO, not AHB like STM32F1
```

Apply fix manually (AI won't auto-edit), re-flash, done.

---

## Step 10: Step-through debug with GDB

```
❯ /build debug        # compile with -O0 -g3 for step-through
❯ /flash
```

Then:
```
(exit TUI with Ctrl+C)
unitymbed debug
```

You're now in GDB with breakpoint at `main`:
```
(gdb) break 15         # breakpoint at line 15
(gdb) continue
(gdb) print GPIOB->POD # read pin state
(gdb) next             # step one line
(gdb) quit
```

---

## What just happened / เกิดอะไรขึ้น

You went from empty folder → blinking LED + UART logging + sensor driver in under 10 minutes, with:
- **0 lines** of HAL boilerplate
- **0 GUI clicks** — everything in terminal
- **Full bare-metal** — 342-byte firmware with direct register access
- **AI grounding** — every prompt sees your `CONTEXT.md`

---

## Next steps

- **Save lessons as you learn**: `/remember <fact>` writes to CONTEXT.md
- **Explore more drivers**: `unitymbed lib list`
- **Go read-only**: `/view src/main.c` to inspect generated code
- **Edit manually**: `/edit src/main.c` opens nano/vim
- **Ship production build**: `unitymbed build` (without `debug`) → 4x smaller binary

---

## Troubleshooting

### "Not inside a UnityMbed project"
You're not in the project folder. `cd` into the directory with `unitymbed.json`.

### "undefined reference to `main'"
`src/main.c` is empty. Re-init or restore from git.

### LED doesn't blink after flash (but flash succeeded)
→ Unplug CMSIS-DAP USB + power-cycle board (SWD keeps CPU in debug state).  
→ Still not working? `unitymbed check` to see register state.

### "quota exceeded"
You've used all Cloud AI requests this month. Options:
- Switch to Ollama: edit `~/.unitymbed/config.yaml` → `provider: ollama`
- Upgrade plan: `unitymbed credits buy cloud_100`

---

**Happy hacking!** Questions: support@unitymbed.com
