# ThinkPad L450 BIOS Firmware Reverse Engineering Report
**Target Motherboard:** Compal AIVL1 NM-A351 (Intel Broadwell i5-5300U / Lenovo IT8586E EC)  
**Status:** Phase 5 (EC Interface Tracking) & Phase 7 (Boot Process Sequence) Complete

---

##  SYSTEM OVERVIEW 
This repository documents the structural analysis, extraction pipeline, and hardware mapping derived from a raw SPI flash dump (`_thinkpad_bios.bin`) of the Lenovo ThinkPad L450 laptop. 

The primary goal of this stage was to decode how the **Intel Broadwell PCH** coordinates with the **ITE IT8586E Embedded Controller (EC)** over the Low Pin Count (LPC) bus to manage power rails, sleep boundaries, thermal sensors, and wake parameters.

---

##  ARCHITECTURAL & DISASSEMBLY PIPELINE 

### 1. SPI Image Mapping
Initial structural analysis was conducted using `binwalk` to chart the partition layout of the raw image binary. A primary 8MB Firmware Volume block was identified at physical address offset `600260`.

### 2. DXE Core Extraction
Using physical offsets carved from the image layout map, we manually isolated individual operational segments using standard binary carvers:
```bash
dd if=600260 of=module_4244.exe bs=1 skip=4244 count=50000

```

* **Result:** Isolated `module_4244.exe`, identified via string analysis as **`DxeMain`** (the Core DXE Foundation Dispatcher). This binary establishes the core runtime structures (`BOOTSERV`, `DXE_SERV`, `RUNTSERV`) necessary to launch secondary hardware drivers.

### 3. ACPI Table Recovery (AML/ASL Mapping)

Because standard PE/COFF `.exe` searches do not track compiled ACPI configuration tables, raw binary pattern queries were run to find physical byte coordinates for the **DSDT** and **SSDT** tables:

```bash
grep -a -b -o "DSDT" 600260

```

* **Target Located:** `2421952` (Absolute physical file offset).
* **Extraction:**
```bash
dd if=600260 of=thinkpad_dsdt.aml bs=1 skip=2421952 count=150000

```


* **Decompilation:** Processed using Intel's ACPI Source Language disassembler (`iasl`) to output a human-readable ASL source layout (`thinkpad_dsdt.dsl`):
```bash
iasl -d thinkpad_dsdt.aml

```



---

##  MAJOR DISCOVERIES & HARDWARE MAPS 

### 1. ITE IT8586E Embedded Controller Memory Layout

The BIOS exposes a 256-byte (`0x0100`) window into the Embedded Controller's internal operational RAM over the LPC bus. This region is explicitly bounded via the `ECOR` (Embedded Control Operational Region) namespace starting at `Device (EC)` (Line 4141):

```asl
OperationRegion (ECOR, EmbeddedControl, 0x00, 0x0100)
Field (ECOR, ByteAcc, NoLock, Preserve) { ... }

```

High-value exposed hardware register mappings discovered:

* **`HFNE` (Byte 0, Bit 3):** Fan Engine/Hardware Interrupt Enable.
* **`HFNS` (Offset `0x0E`):** Dynamic Fan Speed State monitor.
* **`HAM0` - `HAMF` (Offsets `0x10` to `0x1F`):** 16-byte ThinkPad Audio/Mute Matrix and Hardware Asset Management workspace.
* **`HT0H`, `HT0L` / `HT1H`, `HT1L` (Offset `0x2A` onward):** High/Low split byte matrix reading the physical motherboard CPU/Skin Thermal Sensor pins.
* **`HWFN` / `HWBT` (Offset `0x32`):** Hardware Fan Status tracking and Battery Wake/Trip alert lines.

---

##  POWER MANAGEMENT STATE MACHINE 

### PCH Sleep Register Bit Literals

The system parses strict ACPI packages to decide which values must be shifted into the Intel Broadwell PCH's Power Management Controller (`PM1_CNT` register) to trip physical voltage rails down:

| ACPI State | Target Mode | PCH `SLP_TYP` Bit Value | Hardware Behavior |
| --- | --- | --- | --- |
| **`_S3`** | Suspend to RAM | `0x05` | PCH asserts `SLP_S3#` low. System RAM drops to self-refresh state. Main power rails cut. |
| **`_S4`** | Suspend to Disk | `0x06` | PCH dumps memory image to disk block. Core rails cut. |
| **`_S5`** | Soft Off (Shut Down) | `0x07` | PCH asserts `SLP_S5#` low. All system switching rails killed. Only standby auxiliary power lines (`+3VS5`/`+5VS5` for EC monitoring) stay hot. |

---

##  HARDWARE SEQUENCING LIFECYCLE 

### Sleep Entry Phase (`_PTS` - Prepare To Sleep)

When a low-power mode transition event fires, `Method (\_PTS, 1)` takes target state array inputs (`Arg0`) and executes direct board manipulation:

1. **Port 80 Post Output:** Flashes `D80P = Arg0` directly to physical motherboard hardware debug pins for logic verification.
2. **Thermal Protection Safe-Stop:** If dropped into standby/S1 (`Arg0 == 0x01`), the engine backs up fan status values to a global memory block (`\FNID`) and immediately clears both `HFNI` and `HFSP` registers to zero to kill spin velocity before rails collapse.
3. **Hardware Power Trapping:** For S3 drops (`Arg0 == 0x03`), calls an explicit `\TRAP ()` execution branch and snapshots whether the system is on power brick or active battery lines using `_SB.PCI0.LPC.EC.AC._PSR ()`.

### Resume/Wake Phase (`_WAK` - Wake)

Upon PCH power rail recovery, `Method (\_WAK, 1)` processes inverse stabilization hooks:

1. **Event Clearing:** Instantly unmutes physical audio lines (`\_SB.PCI0.LPC.EC.HCMU = 0x00`) and restarts the hardware polling loop via `\_SB.PCI0.LPC.EC.EVNT (0x01)`.
2. **Thermal Spike Override:** Evaluates the `\SCRM` emergency flag block. If active on boot, it instantly forces the EC fan velocity register `HFSP` to `0x07` (Maximum Safety RPM) to prevent thermal lockups before OS microcode scheduling begins.
3. **Hardware Button Clearing:** Scans the 16-bit status register `HWAK` inside the IT8586E. If bit `0x02` is flagged high (indicating physical power button trip or lid trigger), it validates the event cycle and safely zeroes it out (`\_SB.PCI0.LPC.EC.HWAK = 0x00`).

---

##  REPOSITORY ARCHIVE CONTENTS 

* `/codeGetExtractPath/600260` - Raw carved 8MB Firmware Volume bin image.
* `/codeGetExtractPath/module_4244.exe` - Carved x86-64 `DxeMain` module.
* `/codeGetExtractPath/thinkpad_dsdt.aml` - Raw ACPI binary block slice.
* `/codeGetExtractPath/thinkpad_dsdt.dsl` - Fully decompiled, human-readable ASL schematic source text.
* `/codeGetExtractPath/lenovo_ssdt1.aml` + `.dsl` - Carved + decompiled Lenovo SSDT1 (56 B stub).
* `/codeGetExtractPath/lenovo_ssdt2.aml` + `.dsl` - Carved + decompiled Lenovo SSDT2 (LCD backlight `_BCL/_BCM/_BQC`, iGPU + dGPU).
* `/acpi-hotkey.md` - SSDT analysis + complete `HKEY` (`LEN0068`) method reference and EC query → hotkey event dispatch table.
* `/acpi-ssdt.md` - Full 25-SSDT inventory (carves + roles): SgPeg/SgPch, MeSsdt, IFFS (INT3392), ISCT (INT33A0), SATA (SAT1, 5 ports), SaSsdt (GFX/B0D3), DPTF (DptfTa/Ff/La), CPPC, PmRef/Cpu power states, configurable TDP, TPM (dTPM2/MFT0101). Sources in `codeGetExtractPath/_thinkpad_bios.bin.extracted/ssdt/`.
* `/ec-registers.md` - **Definitive 134-register IT8586E EC map** reconstructed from all 7 `Field (ECOR)` blocks (byte.bit offsets: `HDBM`…`PWMH/PWML`).
* `/efi-modules.md` - **376 PE32+ module harvest** (`efi_pe/mNNN.efi` + `efi_manifest.txt` + `efi_catalog.json`): DxeMain, Phoenix (14) / Lenovo (6) fingerprints, VBT/GPU, ACPI-AML+NVRAM, SPI/SMM, SATA/RAID bundles.
* `codeGetExtractPath/_thinkpad_bios.bin.extracted/ssdt/` - all carved SATA/DPTF/CPPC/PM/TPM SSDT `.aml` + `.dsl`.