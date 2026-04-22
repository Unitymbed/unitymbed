# Install guide — ติดตั้ง

Full setup for **macOS**, **Linux**, **Windows**.

---

## macOS (Apple Silicon or Intel)

### 1. Install UnityMbed CLI

```bash
brew tap Unitymbed/tap
brew install unitymbed
```

Verify:
```bash
unitymbed --version   # → unitymbed v0.2.x
```

### 2. Install ARM toolchain + OpenOCD

```bash
brew install --cask gcc-arm-embedded
brew install open-ocd
```

Verify:
```bash
arm-none-eabi-gcc --version
openocd --version
```

### 3. (Optional) Ollama for local AI

```bash
brew install ollama
ollama serve &
ollama pull qwen2.5-coder:7b
```

### 4. Done — test

```bash
unitymbed signup you@example.com
# → check inbox → activate
unitymbed init hello --mcu n32g031
cd hello
unitymbed build
```

---

## Linux (Ubuntu/Debian)

### 1. Install Homebrew on Linux

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Then follow on-screen instructions to add Linuxbrew to PATH
```

Or use the **direct binary** method below if you don't want brew.

### 2. Install UnityMbed

```bash
brew tap Unitymbed/tap
brew install unitymbed
```

**OR** direct binary:
```bash
curl -L https://github.com/Unitymbed/homebrew-tap/releases/latest/download/unitymbed-linux-x64 \
  -o ~/.local/bin/unitymbed
chmod +x ~/.local/bin/unitymbed
# Make sure ~/.local/bin is in your PATH
```

### 3. ARM toolchain + OpenOCD

**Debian/Ubuntu:**
```bash
sudo apt install gcc-arm-none-eabi openocd
```

**Arch:**
```bash
sudo pacman -S arm-none-eabi-gcc arm-none-eabi-newlib openocd
```

**Fedora:**
```bash
sudo dnf install arm-none-eabi-gcc-cs arm-none-eabi-newlib openocd
```

### 4. USB permissions for CMSIS-DAP probe

Create udev rule so you don't need `sudo` to flash:

```bash
sudo tee /etc/udev/rules.d/99-cmsis-dap.rules <<'EOF'
# CMSIS-DAP generic
ATTRS{idVendor}=="0d28", ATTRS{idProduct}=="0204", MODE="0666"
# CMSIS-DAP-JYD (the one in most Nations kits)
ATTRS{idVendor}=="19f5", ATTRS{idProduct}=="3106", MODE="0666"
EOF

sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Windows

### 1. Download binary

Download the latest `.exe`:
```
https://github.com/Unitymbed/homebrew-tap/releases/latest/download/unitymbed-win-x64.exe
```

Save to a folder on your PATH, e.g. `C:\Users\<you>\bin\unitymbed.exe`.

Or via PowerShell:
```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\bin" | Out-Null
Invoke-WebRequest `
  -Uri "https://github.com/Unitymbed/homebrew-tap/releases/latest/download/unitymbed-win-x64.exe" `
  -OutFile "$env:USERPROFILE\bin\unitymbed.exe"
# Add to PATH (one-time)
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$env:USERPROFILE\bin", "User")
```

Open a **new** PowerShell after setting PATH, then:
```powershell
unitymbed --version
```

### 2. ARM toolchain

Download installer from ARM:
```
https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads
```

Choose **arm-gnu-toolchain-X.Y-mingw-w64-i686-arm-none-eabi.exe**. During install, **check "Add to PATH"**.

Verify: `arm-none-eabi-gcc --version` in a new PowerShell.

### 3. OpenOCD

Easiest: use [xpack OpenOCD binary](https://github.com/xpack-dev-tools/openocd-xpack/releases/latest).

```powershell
# download + extract to C:\tools\openocd-0.12.0
# add C:\tools\openocd-0.12.0\bin to PATH
```

### 4. CMSIS-DAP driver (Windows only)

Install the generic CMSIS-DAP driver if Windows doesn't recognize your probe:
https://www.nxp.com/docs/en/application-note-software/CMSIS-DAP-WIN-DRIVER.zip

Or use [Zadig](https://zadig.akeo.ie/) to replace the driver with **WinUSB** for your probe.

### 5. Make + bash (for Makefile-based build)

Install [Git for Windows](https://git-scm.com/download/win) — gives you Git Bash with `make` included.

Run UnityMbed from **Git Bash**, not PowerShell, for best compatibility.

---

## Verify full setup (all platforms)

```bash
unitymbed setup
```

Expected output:
```
UnityMbed setup — checking dependencies…

  ✓ arm-none-eabi-gcc     /opt/homebrew/bin/arm-none-eabi-gcc
  ✓ openocd               /opt/homebrew/bin/openocd
  ✓ make                  /usr/bin/make
  ✓ ollama                /opt/homebrew/bin/ollama

All dependencies present.
```

---

## Hardware recommendations / ฮาร์ดแวร์แนะนำ

| Item | Why | Price |
|------|-----|-------|
| **N32G031K8U7 dev board** | Target MCU | ~50-100฿ |
| **CMSIS-DAP-JYD probe** (or DAPLink) | SWD flash + debug | ~300-500฿ |
| **USB-to-TTL converter** (CP2102) | UART monitor (optional — many probes have VCP built-in) | ~80-150฿ |
| **Jumper wires** (female-to-female) | Connect probe to board | ~50฿ |

Tested combos:
- CMSIS-DAP-JYD + N32G031 dev board — works out of the box
- DAPLink v2 + N32G031 — works with our custom OpenOCD config

---

## Troubleshooting

### `openocd: unable to find a matching CMSIS-DAP device`
Probe not plugged in, or USB permissions (Linux). Run `ioreg -p IOUSB -c IOUSBHostDevice` (Mac) or `lsusb` (Linux) to verify detection.

### Build fails: `undefined reference to main`
Your `src/main.c` got emptied. Re-init or restore via `unitymbed lib add template`.

### LED doesn't blink after flash
Most common cause: GPIO clock not enabled. N32G031 uses **APB2PCLKEN** not AHBPCLKEN. Run `unitymbed check` to diagnose.

### Flash succeeds but CPU halted
Remove SWD probe USB + power-cycle the board. SWD connection keeps CPU in debug state until disconnected.

---

## Next

- [Tutorial — blink + UART in 10 min](tutorial.md)
- [Command reference](commands.md)
- [AI setup](ai.md)
