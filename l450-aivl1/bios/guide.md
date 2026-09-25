# ThinkPad L450 (NM-A351) — BIOS Firmware Reverse Engineering

> **Board:** Lenovo ThinkPad L450 | **Schematic Ref:** AIVL1 NM-A351  
> **Firmware Version:** R06ET69W (v1.31) — `JDET69WW/$0AJD000.FL1`   

---

## Table of Contents

- [ThinkPad L450 (NM-A351) — BIOS Firmware Reverse Engineering](#thinkpad-l450-nm-a351--bios-firmware-reverse-engineering)
  - [Table of Contents](#table-of-contents)
  - [Project Goals](#project-goals)
  - [Hardware Component Inventory](#hardware-component-inventory)
  - [Tool Setup](#tool-setup)
  - [Stage 1 — Firmware Extraction](#stage-1--firmware-extraction)
    - [Source](#source)
    - [Extraction](#extraction)
    - [Key File](#key-file)
  - [Stage 2 — Firmware Structure Analysis](#stage-2--firmware-structure-analysis)
    - [Initial binwalk scan of the capsule wrapper](#initial-binwalk-scan-of-the-capsule-wrapper)
    - [Extracting the raw payload](#extracting-the-raw-payload)
  - [Stage 3 — UEFI Volume Dissection](#stage-3--uefi-volume-dissection)
    - [binwalk scan of the raw image](#binwalk-scan-of-the-raw-image)
    - [Notable: No Intel ME Region](#notable-no-intel-me-region)
    - [Inspecting the DxeMain module](#inspecting-the-dxemain-module)
    - [String survey of the full firmware image](#string-survey-of-the-full-firmware-image)
  - [Stage 4 — ACPI Table Extraction \& Decompilation](#stage-4--acpi-table-extraction--decompilation)
    - [Locating ACPI table signatures in the raw binary](#locating-acpi-table-signatures-in-the-raw-binary)
    - [Carving and decompiling the DSDT](#carving-and-decompiling-the-dsdt)
  - [Stage 5 — Embedded Controller (EC) Interface Map](#stage-5--embedded-controller-ec-interface-map)
    - [EC Device Declaration](#ec-device-declaration)
    - [EC Register Map (IT8586E — ECOR field layout)](#ec-register-map-it8586e--ecor-field-layout)
  - [Stage 6 — Power State Machine Reconstruction](#stage-6--power-state-machine-reconstruction)
    - [ACPI Global Sleep State Packages](#acpi-global-sleep-state-packages)
    - [`_PTS` — Prepare To Sleep (line 14528)](#_pts--prepare-to-sleep-line-14528)
    - [`_WAK` — Wake From Sleep (line 14666)](#_wak--wake-from-sleep-line-14666)
    - [Full Power Lifecycle Diagram](#full-power-lifecycle-diagram)
  - [Key Findings Summary](#key-findings-summary)
  - [File Inventory](#file-inventory)
  - [Might-TODO](#might-todo)

---

## Project Goals

This project reverse engineers the firmware and hardware interface of the Lenovo ThinkPad L450 motherboard (NM-A351) from software alone — no physical chip programmer required. The three candidate end-goals are:

| Goal | Description |
|---|---|
| **Open-Source Schematic** | Produce a public hardware reference (KiCad block diagram, power rail map, repair guide) for the right-to-repair community |
| **Coreboot / Open Firmware** | Assess feasibility of an open-source BIOS port; identify Intel ME region and neutralize with `me_cleaner` |
| **Hardware Telemetry Dashboard** | Map EC register offsets well enough to read raw battery, thermal, and fan data without an OS |

---

## Hardware Component Inventory

Key ICs identified on the NM-A351 motherboard:

| Marking / Part | True Identity | Function |
|---|---|---|
| `WINBOND 25064FVS10` | Winbond W25Q64FV (64Mbit) | **BIOS Flash Chip** — 8MB SPI NOR flash; holds all UEFI firmware |
| `GL8506` | Genesis Logic GL850G | USB 2.0 Hub Controller — splits one upstream USB port into multiple internal/external ports |
| `PI3L 720ZHE` | Pericom PI3L720 | Gigabit Ethernet mux — switches differential pairs for the LAN port |
| `PS8338B` | Parade Technologies PS8338B | DisplayPort demux — routes video signals from Intel HD Graphics to LCD or mini-DP/dock |
| `80 24780 I3` | Texas Instruments BQ24780S | **Battery Charge Controller** — manages 20V DC-in path, switching between battery/wall, drives charging buck converter |
| `IT8586E` | ITE IT8586E | **Embedded Controller (EC)** — handles keyboard, fan, thermal, battery, power button, LPC bus |
| `NCP81101` | ON Semi NCP81101 | CPU VCore buck converter — steps voltage down for the Intel i5-5300U (Broadwell-U) |

---

## Tool Setup

```bash
# Install all required tools
sudo apt update
sudo apt install innoextract binwalk acpica-tools uefitool-cli xxd

# Note: uefitool-cli (v0.28.0) only ships UEFIPatch and UEFIReplace on Ubuntu 22.04
# UEFIExtract binary is NOT included — use binwalk as fallback for extraction
```

---

## Stage 1 — Firmware Extraction

### Source

Downloaded the official Lenovo BIOS update utility from Lenovo Support:
- **File:** `bios_dump.exe` — Windows self-extracting InnoSetup package
- **Firmware version:** `R06ET69W` (v1.31)

### Extraction

```bash
innoextract bios_dump.exe
```

**Output:**
```
Extracting "version 1.31 (R06ET69W)" - setup data version 5.5.7 (unicode)
 - "codeGetExtractPath/WINUPTP.EXE"
 - "codeGetExtractPath/WinFlash64.exe"
 - "codeGetExtractPath/JDET69WW/$0AJD000.FL1"
 ... (flasher utilities, logo tools, driver helpers)
```

### Key File

```
JDET69WW/$0AJD000.FL1   ← Raw BIOS binary (Lenovo uses .FL1/.FL2 extension)
```

```bash
# Copy and rename for standard tooling
cp JDET69WW/\$0AJD000.FL1 ./thinkpad_bios.bin
```

---

## Stage 2 — Firmware Structure Analysis

### Initial binwalk scan of the capsule wrapper

```bash
binwalk thinkpad_bios.bin
```

**Notable findings:**

| Offset (Dec) | Offset (Hex) | Description |
|---|---|---|
| 0 | `0x0` | EFI Capsule v0.9 — signed update wrapper |
| 80 | `0x50` | UEFI PI Firmware Volume (FFS v2) |
| 6,292,064 | `0x600260` | **LZMA compressed payload** — uncompressed size: **8,212,480 bytes (~8MB)** |
| 10,429,416+ | `0x9F23E8+` | Intel x86/x64 microcode blobs (Broadwell signatures: `0x306d4`, `0x40651`) |
| 10,561,858+ | various | x509v3 certificates (Secure Boot chain) |
| 10,814,254 | `0xA5032E` | GIF image: Lenovo boot logo (600×260) |
| 13,126,336 | `0xC84AC0` | Copyright string: Lenovo Group Limited |

**Key insight:** The `.FL1` file is a **UEFI Capsule Update**, not a raw chip dump. The actual 8MB flash image is LZMA-compressed inside it at offset `0x600260`.

### Extracting the raw payload

```bash
binwalk -e thinkpad_bios.bin
# Produces: _thinkpad_bios.bin.extracted/

ls -lh _thinkpad_bios.bin.extracted/
# 600260      7.9M   ← uncompressed raw BIOS image
# 600260.7z   6.7M   ← compressed archive (source)
```

The file `600260` (7.9MB) is the **true motherboard flash image** — byte-for-byte equivalent to what is written to the Winbond W25Q64FV chip.

---

## Stage 3 — UEFI Volume Dissection

### binwalk scan of the raw image

```bash
cd _thinkpad_bios.bin.extracted/
binwalk 600260
```

**Structure overview:**

| Offset (Dec) | Offset (Hex) | Description |
|---|---|---|
| 4,096 | `0x1000` | UEFI PI Firmware Volume (FFS v2) — main BIOS region |
| 4,244 | `0x1094` | First PE32 module — **DxeMain** (DXE Foundation Dispatcher) |
| 289,082 | `0x4693A` | Copyright: Intel Corp. 2000–2011 |
| 2,600,086 | `0x27AC96` | Copyright: Intel Corporation 1997–2013 |
| 2,973,937 | `0x2D60F1` | GIF — Lenovo boot logo (546×87) |
| 6,693,451 | `0x66224B` | Phoenix EDK2 Network source path string |
| 6,836,085 | `0x684F75` | **`\Phoenix\Edk2Network\000\NetworkPkg\Ip6Dxe\Ip6Nd.c`** — confirms Phoenix EDK2 codebase |
| 7,601,696 | `0x73FE20` | GIF image data (boot animation assets) |
| 7,791,132 | `0x76E21C` | Copyright: Phoenix Technologies Ltd. 2007–2010 |
| 8,062,290 | `0x7B0552` | Copyright: Phoenix Technologies Ltd. 1985–2013 |

**Confirmed:** This firmware is built on a **Phoenix Technologies EDK2 codebase** (EFI Development Kit II), customized by Lenovo.

### Notable: No Intel ME Region

This UEFI capsule update intentionally **excludes** the Intel ME (Management Engine) and Flash Descriptor regions — it only reflashes the BIOS region to prevent bricking during standard updates. A physical CH341A dump would be required to capture the full SPI flash including ME and descriptor.

### Inspecting the DxeMain module

```bash
dd if=600260 of=module_4244.exe bs=1 skip=4244 count=50000
file module_4244.exe
# → PE32+ executable (DLL) (EFI boot service driver) x86-64
strings module_4244.exe | head -n 50
```

**Confirmed strings:** `DxeMain`, `CoreInitializeDispatcher`, `CoreDispatcher`, `BOOTSERV`, `DXE_SERV`, `RUNTSERV`, `Real Time Clock`, `Watchdog Timer`, `Metronome`, `Capsule`, `Variable Write`

This is the **DXE Foundation** — the core dispatcher that launches all hardware initialization drivers on boot.

### String survey of the full firmware image

```bash
strings 600260 > all_firmware_strings.txt

grep -iE "Lenovo|Lnv|Tpad" all_firmware_strings.txt
```

**Key Lenovo-specific identifiers found:**

```
WLENOVOTP-SSDT1          ← Lenovo custom ACPI SSDT table #1
LENOVOTP-SSDT2           ← Lenovo custom ACPI SSDT table #2
LenovoPreSave
LenovoBDG
LenovoScratchData
LenovoSystemConfig
LenovoConfig
LenovoSecurityConfig
LenovoFunctionConfig
LenovoHiddenSetting
LenovoSavedDefault
(C) COPYRIGHT LENOVO 2005, 2014 ALL RIGHTS RESERVED
```

---

## Stage 4 — ACPI Table Extraction & Decompilation

### Locating ACPI table signatures in the raw binary

Using `grep -b` on the raw binary (not on `strings` output, which has shifted offsets):

```bash
grep -a -b -o "DSDT" 600260
grep -a -b -o "SSDT" 600260
```

**DSDT candidates (absolute byte offsets):**

```
797345    2421952    3165022    4007903    5098202    5638380
7095037   7099875    7101279    7101451    7114321
```

**Target:** Offset `2421952` — verified with `xxd` as a valid ACPI DSDT header.

### Carving and decompiling the DSDT

```bash
dd if=600260 of=thinkpad_dsdt.aml bs=1 skip=2421952 count=150000

iasl -d thinkpad_dsdt.aml
# Output: thinkpad_dsdt.dsl (553,120 bytes of human-readable ASL source)
```

**DSDT header:**
```
ACPI: DSDT 0x00000000 0111E5 (v02 IBM TP-JD 00001310 INTL 20120711)
```

- OEM ID: `IBM` (legacy Lenovo/IBM attribution)
- Table ID: `TP-JD` (ThinkPad JD — internal Lenovo board codename)
- Compiler: Intel ACPI CA 20120711

---

## Stage 5 — Embedded Controller (EC) Interface Map

### EC Device Declaration

Found at **line 4141** of `thinkpad_dsdt.dsl`:

```asl
Device (EC)
{
    Name (_HID, EisaId ("PNP0C09"))  // Embedded Controller Device
    Name (_UID, 0x00)
    Name (_GPE, 0x25)                // General Purpose Event #37

    OperationRegion (ECOR, EmbeddedControl, 0x00, 0x0100)
    Field (ECOR, ByteAcc, NoLock, Preserve) { ... }
}
```

The EC occupies a **256-byte (0x100) address space** in EmbeddedControl mode. All registers are accessed by the BIOS via the LPC bus at the standard EC I/O ports (0x62 / 0x66).

### EC Register Map (IT8586E — ECOR field layout)

| Register Name | Byte Offset | Bit(s) | Description |
|---|---|---|---|
| `HDBM` | 0x00 | bit 0 | Hardware Debug Mode flag |
| `HKLK` | 0x00 | bit 2 | Hardware Keyboard Lock |
| `HFNE` | 0x00 | bit 3 | **Fan Enable** toggle |
| `HLDM` | 0x00 | bit 6 | Hardware Lid Mode |
| `BBLS` | 0x01 | bit 0 | Battery/Backlight state |
| `BTCM` | 0x01 | bit 1 | Battery Current Mode |
| `HBPR` | 0x01 | bit 5 | Hardware Battery Present |
| `BTPC` | 0x01 | bit 6 | Battery Pack Charging |
| `HDUE` | 0x02 | bit 0 | Hardware Device USB Enable |
| `SNLK` | 0x02 | bit 5 | Scroll/NumLock state |
| `HAUM` | 0x03 | bits 5-6 | Hardware Audio Mute mode |
| `HSPA` | 0x05 | bit 0 | Hardware Suspend/Power Active |
| `HSUN` | 0x06 | 8-bit | Hardware Suspend notification |
| `HSRP` | 0x07 | 8-bit | Hardware Suspend/Resume Phase |
| `FNKS` | 0x09 | bit 6 | Function Key Status |
| `HLCL` | 0x0C | 8-bit | Hardware LCD Level (brightness) |
| `CALM` | 0x0D | bit 4 | Camera Lock Mode |
| `HFNS` | 0x0E | bits 0-1 | **Fan Speed State** (2-bit mode index) |
| `NULS` | 0x0F | bit 6 | Num/Caps Lock state |
| `HAM0`–`HAMF` | 0x10–0x1F | 8-bit × 16 | Hardware Asset Management / Audio Matrix (16 bytes) |
| `HANT` | 0x23 | 8-bit | Hardware Antenna state |
| `HANA` | 0x26 | bits 2-3 | Hardware Antenna mode |
| `SKEM` | 0x28 | bit 1 | Skew/EMI mode |
| `HATR` | 0x2A | 8-bit | Hardware ATR (thermal reference) |
| `HT0H` / `HT0L` | 0x2B / 0x2C | 8-bit | **Thermal Sensor 0 — High/Low bytes** |
| `HT1H` / `HT1L` | 0x2D / 0x2E | 8-bit | **Thermal Sensor 1 — High/Low bytes** |
| `HFSP` | 0x2F | 8-bit | **Fan Speed** (PWM target, 0x00=off, 0x07=max) |
| `HMUT` | 0x30 | bit 6 | Hardware Mute (audio) |
| `HUWB` | 0x31 | bit 2 | Hardware UWB/Wireless module |
| `HWPM` | 0x32 | bit 0 | Hardware Wake — Power Button |
| `HWLB` | 0x32 | bit 1 | Hardware Wake — Lid Open |
| `HWLO` | 0x32 | bit 2 | Hardware Wake — Lid state |
| `HWDK` | 0x32 | bit 3 | Hardware Wake — Dock event |
| `HWFN` | 0x32 | bit 4 | Hardware Wake — Fan event |
| `HWBT` | 0x32 | bit 5 | Hardware Wake — Battery event |
| `HWAK` | 0x??? (4297) | 16-bit | **Wake event register** — cleared to `0x00` after wake |

---

## Stage 6 — Power State Machine Reconstruction

### ACPI Global Sleep State Packages

Found at lines **14507–14525** of `thinkpad_dsdt.dsl`:

```asl
Name (\_S3, Package (0x04) { 0x05, 0x05, 0x00, 0x00 })  // S3: Suspend to RAM
Name (\_S4, Package (0x04) { 0x06, 0x06, 0x00, 0x00 })  // S4: Hibernate
Name (\_S5, Package (0x04) { 0x07, 0x07, 0x00, 0x00 })  // S5: Soft Power Off
```

These values are written to the **Intel Broadwell PCH PM1_CNT (Power Management Control)** register to trigger hardware sleep transitions:

| State | Mode | PCH `SLP_TYP` Value | Effect |
|---|---|---|---|
| `_S3` | Suspend to RAM | `0x05` | Cuts CPU/peripheral power; RAM enters self-refresh |
| `_S4` | Hibernate | `0x06` | RAM image written to disk; near-full power cut |
| `_S5` | Soft Off | `0x07` | Full shutdown; only +3VS5/+5VS5 standby rails remain active |

### `_PTS` — Prepare To Sleep (line 14528)

Called by the OS before any sleep/off transition. `Arg0` = target sleep state (e.g., `0x03` for S3).

```asl
Method (\_PTS, 1, NotSerialized)
{
    D80P = Arg0                          // POST code output to physical Port 80 debug pins
    \SPS = Arg0                          // Store current sleep state in global variable

    // Fan cutoff (S1 state only)
    \FNID = \_SB.PCI0.LPC.EC.HFNI       // Save current fan index
    \_SB.PCI0.LPC.EC.HFNI = 0x00        // Stop fan index
    \_SB.PCI0.LPC.EC.HFSP = 0x00        // ← WRITE 0x00 to HFSP: fans stop completely

    // S3 path
    \VVPD (0x03)                         // Platform video power-down
    \TRAP ()                             // Lock execution boundaries (SMM trap)
    \ACST = \_SB.PCI0.LPC.EC.AC._PSR () // Check AC adapter presence before sleep

    // Disable wake-on-fan if not configured
    \_SB.PCI0.LPC.EC.HWFN = 0x00        // Clear wake-on-fan flag in EC

    \_SB.PCI0.LPC.EC.HKEY.MHKE (0x00)   // Disable HotKey EC events
}
```

**Hardware trace:** `D80P` writes the sleep state value directly to Port 80 — this is the same port the diagnostic LED card reads during POST, meaning engineers can see exactly which sleep state was being entered at power failure time.

### `_WAK` — Wake From Sleep (line 14666)

Called by the OS immediately after the system resumes power delivery.

```asl
Method (\_WAK, 1, NotSerialized)
{
    D80P = (Arg0 << 0x04)                    // Output wake state to Port 80 (shifted)

    \SPS = 0x00                              // Clear sleep state variable
    \_SB.PCI0.LPC.EC.HCMU = 0x00            // Unmute audio hardware
    \_SB.PCI0.LPC.EC.EVNT (0x01)            // Trigger EC hardware event refresh
    \_SB.PCI0.LPC.EC.HKEY.MHKE (0x01)       // Re-enable HotKey EC events
    \_SB.PCI0.LPC.EC.FNST ()                // Restore function key state

    // Restore fan speed (emergency thermal protection)
    If (\SCRM) {
        \_SB.PCI0.LPC.EC.HFSP = 0x07        // ← WRITE 0x07: max fan speed on wake
    }

    // Check if AC state changed during sleep
    If ((\ACST != \_SB.PCI0.LPC.EC.AC._PSR ())) {
        \_SB.PCI0.LPC.EC.ATMC ()            // Trigger AC transition management
    }

    // Wake register cleanup
    \_SB.PCI0.LPC.EC.HWAK = 0x00            // Clear 16-bit wake event register in EC
}
```

**Wake event detection:** The EC's `HWAK` register (16-bit) is checked for bit `0x02` to classify the wake source (power button, lid, dock, etc.). After handling, it is explicitly cleared to prevent stale events on the next cycle.

### Full Power Lifecycle Diagram

```
[OS Triggers Sleep / Shutdown]
           │
           ▼
1. Method (\_PTS, Arg0)
   ├── D80P = Arg0              → Post code to Port 80 physical debug pins
   ├── HFSP = 0x00              → EC cuts fan PWM (fans spin down)
   ├── HFNI = 0x00              → EC clears fan index register
   ├── \TRAP()                  → SMM trap locks execution boundaries
   └── ACST = AC._PSR()         → Records whether on battery or wall power
           │
           ▼
2. OS reads Name (\_Sx) package
   └── Fetches SLP_TYP value (S3→0x05, S4→0x06, S5→0x07)
           │
           ▼
3. OS writes SLP_TYP to PCH PM1_CNT register
   └── PCH asserts SLP_S3# / SLP_S4# / SLP_S5# hardware pins LOW
           │
           ▼
4. IT8586E EC responds to SLP_S5# rail drop
   └── EC toggles MOSFET power gates → system rails cut

──────────── [POWER RESTORED / WAKE EVENT] ────────────

           │
           ▼
5. Method (\_WAK, Arg0)
   ├── HCMU = 0x00              → Unmute audio lines
   ├── EVNT(0x01)               → Refresh EC hardware event queue
   ├── HFSP = 0x07 (if SCRM)   → Emergency: max fan speed for thermal protection
   ├── ATMC()                   → Handle any AC/battery state change during sleep
   └── HWAK = 0x00              → Clear 16-bit wake event register
```

---

## Key Findings Summary

| Finding | Detail |
|---|---|
| **Firmware codebase** | Phoenix Technologies EDK2 (EFI Development Kit II), heavily customized by Lenovo |
| **DSDT OEM ID** | `IBM` / Table ID `TP-JD` — legacy IBM attribution still present in 2014 board |
| **EC chip** | ITE IT8586E, connected over LPC bus at standard ports 0x62/0x66 |
| **EC address space** | 256 bytes (0x00–0xFF), full register map recovered from `ECOR` field |
| **Thermal sensors** | Two sensors: `HT0H:HT0L` and `HT1H:HT1L` (split high/low byte format) |
| **Fan control register** | `HFSP` at EC offset 0x2F — `0x00` = off, `0x07` = maximum RPM |
| **S3 SLP_TYP value** | `0x05` written to PCH PM1_CNT to enter Suspend-to-RAM |
| **S5 SLP_TYP value** | `0x07` written to PCH PM1_CNT to enter Soft Off |
| **Port 80 debug** | Sleep state is written to Port 80 pins at every `_PTS` call — useful for power failure diagnosis |
| **Wake source detection** | EC `HWAK` (16-bit) tracks wake event type; bit `0x02` = classified wake event |
| **No Intel ME in capsule** | The `.FL1` capsule deliberately excludes ME and descriptor regions |
| **Lenovo custom ACPI** | Two Lenovo SSDTs (`WLENOVOTP-SSDT1`, `LENOVOTP-SSDT2`) contain additional platform methods |
| **HotKey subsystem** | `HKEY` device under EC manages all Fn/hotkey events via `MHKE`/`MHKQ` methods |

---

## File Inventory

```
project root/
├── bios_dump.exe                          # Original Lenovo BIOS update utility (input)
├── codeGetExtractPath/
│   ├── JDET69WW/
│   │   └── $0AJD000.FL1                  # Raw Lenovo BIOS binary (FL1 format)
│   ├── thinkpad_bios.bin                  # Renamed copy of FL1 for tooling
│   └── _thinkpad_bios.bin.extracted/
│       ├── 600260                         # Decompressed 7.9MB raw BIOS image ← MAIN ARTIFACT
│       ├── 600260.7z                      # Compressed source
│       ├── module_4244.exe                # Carved DxeMain PE32+ module
│       ├── acpi_tables.aml                # Carved ACPI region (rough cut — do not use directly)
│       ├── thinkpad_dsdt.aml              # Correctly carved DSDT binary (from offset 2421952)
│       ├── thinkpad_dsdt.dsl              # ★ Decompiled DSDT — 553KB of ASL source (MAIN REFERENCE)
│       └── all_firmware_strings.txt       # Full strings dump of the 600260 image
```

---

## Might-TODO

- [x] **Decompile Lenovo SSDTs** — done in `acpi-hotkey.md`. `TP-SSDT1` @2420732 (56 B, stub `KOU1`), `TP-SSDT2` @2420788 (1158 B, `_BCL`/`_BCM`/`_BQC` backlight for iGPU `VID.LCD0` + dGPU `PEG.VID.LCD0`), both carved + decompiled (`lenovo_ssdt1/2.aml/.dsl`).
- [x] **Map the `HKEY` subsystem** — full `LEN0068` method reference + complete `_Qxx` → `MHKK` bit → `MHKQ` event dispatch table in `acpi-hotkey.md` (incl. queue classification and `MHKP` dequeue order).
- [x] **Reconstruct the EC register map (software)** — exhaustive parse of all 7 `Field (ECOR)` blocks → definitive 134-register byte.bit map in `ec-registers.md` (cross-refs `HT0H/HT1H/HFSP/PWMH/PWML/HUBS` etc. to `_Qxx` handlers and DSDT).
- [x] **Decompile all remaining SSDTs** — 25/25 carved + decompiled, inventory in `acpi-ssdt.md` (SATA/DPTF/CPPC/PmRef/TPM; sources in `codeGetExtractPath/_thinkpad_bios.bin.extracted/ssdt/`).
- [x] **Harvest EFI modules** — 376 PE32+ carved from `600260` (`efi_pe/`), catalog in `efi-modules.md` (Phoenix 14×, Lenovo 6×; main Phoenix DXE volume @0x5d0000–0x7b0000).
- [ ] **Reconstruct the Thermal Zone map** — grep for `ThermalZone` in `thinkpad_dsdt.dsl` and cross-reference with `HT0H`/`HT1H` EC registers; build a table of thermal trip points (soil for the DPTF `\_PR.` dials)
- [ ] **Physical BIOS dump** — acquire a CH341A programmer + SOIC-8 clip and dump the full Winbond W25Q64FV chip to capture the Intel Flash Descriptor and ME region (currently absent from the capsule)
- [ ] **Intel ME analysis** — once the full dump is obtained, use `me_cleaner` to analyze and optionally neutralize the Intel ME region
- [ ] **Power rail tracing** — use the BQ24780S datasheet + ACPI AC path (`_SB.PCI0.LPC.EC.AC._PSR`) findings to trace the 20V DC-in path through input fuses and MOSFETs on the board
- [ ] **Disassemble power management drivers** — identify PE modules in `600260` related to platform power (search for `PlatformPower`, `PchInit` GUIDs) and load into Ghidra for static analysis
- [ ] **Open-source schematic** — assemble KiCad block diagram from EC register map, power rail analysis, and chip datasheet cross-references
- [ ] **Coreboot feasibility study** — compare hardware initialization sequence against existing Coreboot Broadwell-U board ports
- [ ] **EC telemetry reader** — write a Linux userspace tool to read `HFSP`, `HT0H:HT0L`, `HT1H:HT1L`, and battery registers directly from `/dev/ec` or via ACPI sysfs

---