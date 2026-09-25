# ACPI SSDT Inventory — L450 / AIVL1 (R06ET69W)

All 25 SSDTs carved from the raw BIOS image `600260`, decompiled with `iasl 20200925`
(plain `iasl -d`, no `-e`; external references unresolved — cosmetic). Sources in
`ssdt/*.aml` + `ssdt/*.dsl`.

Offsets are absolute file offsets into `_thinkpad_bios.bin.extracted/600260`.

## Pre-DSDT SSDTs (Firmware Volume region, before DSDT)

| File (carve) | Build | OemId/TableId | Size | Role / key findings |
|---|---|---|---|---|
| `SgRef_SgPeg_SgPeg` | INTl-20120711 | SgRef / SgPeg | 520 B @400468 | PEG slot device `\_SB.PCI0.PEG0.PEGP` (NIDS/CMDS), repros `PGOF`/`PGON` (PEG power gating used with AMD2/AMD3 in DSDT). 2 unresolved externals. |
| `SgRef_SgPch_SgPch` | INTl-20120711 | SgRef / SgPch | 1247 B @401020 | PCH root-port `\_SB.PCI0.RP05.PEGP` — `_ADR/_PRW/_ON/_OFF` (PCIe ASPM / wake for the PEG root port used by dGPU), also `Iffs` companion. |
| `MeSsdt_MeSsdt` | — | MeSsdt / ME? | 75 B @1469428 | Stub. |
| `Iffs_IffsAsl` | — | IFFS / IffsAsl | 208 B @1705596 | Device `IFFS`, HID **INT3392** (CID PNP0C02) — Intel companion "motherboard resources" device; occupies an LPC region (SPI/flash access). |
| `IsctTabl_Isct` | — | IsctTabl / Isct | 1532 B @1843860 | Device `IAOE`, HID **INT33A0** = Intel Smart Connect Technology (waits for wake timers, SATA/network resume scheduling). |

## Post-DSDT SSDTs (the 20 tables found after the DSDT, contiguous region)

| File | OemId/TableId | Size/@offset | Role / key findings |
|---|---|---|---|
| `Sata_SataAhci` | SataRe / SataAhci | 2507 B @2492076 | AHCI device `\_SB.PCI0.SAT1` w/ 5 SATA ports `PRT0..PRT4`, each `_ADR/_GTF/_SDD` — SAT1 is 00:1F.2 main controller. 5th port maps the L450's SATA3/M.2 SSB? |
| `Sata_SataPri` | SataRe / SataPri | 2295 B @2494588 | First/second ATA channel config (port 0). |
| `Sata_SataSec` | SataRe / SataSec | 1787 B @2496888 | Secondary ATA config (port 1). |
| `SaSsdt_SaSsdt` | SaSsdt | 5239 B @2498708 | System-Agent SSDT: enables `\_SB.PCI0.VID` (iGPU display), adds `\_SB.PCI0.B0D3` (on-chip 00:03 graphics) with `_INI/_STA`, PCIe lane config. |
| `Dptf_DptfTa` | DptfTa / DptfTabl | 24444 B @2507356 | Intel DPTF (Dynamic Platform & Thermal Framework) — thermal/power policies; reads `\_PR.` CPU/P-state dials (AAC0/ACRT/APSV … PL10/PL11/PL12/PL20/PL21/PL22 power limits) defined in DSDT `_PR_`. |
| `Dptf_DptfFf` | DptfFf / DptfFfrd | 21069 B @2531828 | DPTF fan/thermal framework ("Ffrd") — fan curve params feeding `DPTF` fan device. |
| `Dptf_DptfLa` | DptfLa / DptfLam_ | 18325 B @2552932 | DPTF "LAM/LAT" thermal-plan SSDT (TSK thresholds). |
| `Cpc_CpcTa` | CpcTa / Cpc_Ta | 2756 B @2574812 | CPPC (Collaborative Processor Performance Control) topolo; `_CPC` tables for LPU. |
| `Cpc_CppcTa` | CppcTa / CppcTa | 916 B @2577572 | CPPC device table (HID "CPC"?); `_STA/_DSM` present. |
| `PmRef_ApCst` | PmRef / ApCst | 281 B @2583644 | All-processor C-state bits (`\_PR.CST?`), Ap C-state export. |
| `PmRef_ApTst` | PmRef / ApTst | 840 B @2583932 | Throttle state export for APs. |
| `PmRef_Cpu0Tst` | PmRef / Cpu0Tst | 734 B @2584776 | CPU0 throttle state (`_TSS`). |
| `CpuRef_CpuSsdt` | CpuRef / CpuSsdt | 2932 B @2585516 | `\_PR.CPU0` full processor object — `_HID/_UID` (ACPI0007), `_PPC`, `_PSS`, `_CST`, `_PSD`, `_TPC`, `_OSC`, safe to wire up. |
| `PmRef_ApIst` | PmRef / ApIst | 1450 B @2588452 | P-state (`_PSS`) tables for APs. |
| `PmRef_Cpu0Cst` | PmRef / Cpu0Cst | 1078 B @2589908 | CPU0 `_CST` (C1/C2/C3 with MSR + EC assist?). |
| `PmRef_Cpu0Ist` | PmRef / Cpu0Ist | 1578 B @2590992 | CPU0 `_PSS`/`_PPC`/`_PSD` — P-state full table (P0..Pn). |
| `Ctdp_CtdpB` | CtdpB | 873 B @2592576 | configurable TDP (`_PPC`-gated, `PL*` → EC/DPTF). |
| `PmRef_LakeTiny` | PmRef / LakeTiny | 455 B @2593456 | Low-power "tiny" C-state/LPT handling (SAT0/SAT1 S0ix sleep helpers). |
| `Intel__Tpm` | Intel_ / TpmTable | 1712 B @2680596 | TPM PPI/board device `\_SB.PCI0.LPC` TPM — dynamic `_HID` method. |
| `Intel__Tpm2` | Intel_ / Tpm2Tabl | 1174 B @2682340 | **TPM 2.0** device, HID **MSFT0101** `\_SB` — `_DSM`, `_STR`. L450's dTPM (Infineon SLB9660/9665) exposed here. |

## Summary observations

- Peg/PEG power uses a **3-state scheme**: SgPeg (PEGP) + SgPch (RP05) SSDTs + DSDT
  PowerResources `AMD2`/`AMD3` drive the optional R7-M260 dGPU via `_ON/_OFF` that call
  ACPI `\VHYB`, wait on link via `\LCHK`, with `RTLK` retry-lock. (See `power-tree`.)
- DPTF reads its power/thermal dials entirely from DSDT `\_PR.` scope (`AAC0`=critical °C,
  `ACRT`, `APSV`=passive °C) — the real tunables live in `thinkpad_dsdt.dsl`, not the SSDTs.
- EC sensor regs used from ACPI: `TMP0` (ambient), `TSL0..3` (thermal-sensor low), `HTMH/HTML`,
  `HFSP` fan-speed, `PWMH/PWML` PWM, fan status `HFNE`, `HUBS` USB-hub-power — all now in
  `ec-registers.md`.