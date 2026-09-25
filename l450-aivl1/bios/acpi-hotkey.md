# Lenovo ACPI Extras — SSDTs + HKEY Hotkey Subsystem

**Board:** AIVL1 NM-A351 | **Firmware:** R06ET69W (v1.31) | Source: `600260` raw BIOS image

---

## Table locations (absolute byte offsets in `600260`)

| Table | Offset | Size | ACPI header |
| ----- | ------ | ---- | ----------- |
| SSDT1 (`TP-SSDT1`) | `2420732` | 56 B | v01 LENOVO TP-SSDT1 rev 0x0100, INTL 20120711 |
| SSDT2 (`TP-SSDT2`) | `2420788` | 1158 B | v01 LENOVO TP-SSDT2 rev 0x0200, INTL 20120711 |
| DSDT (`TP-JD`) | `2421952` | 70117 B | v02 IBM TP-JD, INTL 20120711 |

All checksums verify. Tables stored contiguously; SSDT2 ends at `2421946`, followed by `6` pad bytes before the DSDT.

Carved + decompiled with:
```bash
dd if=600260 of=lenovo_ssdt1.aml bs=1 skip=2420732 count=56
dd if=600260 of=lenovo_ssdt2.aml bs=1 skip=2420788 count=1158
iasl -d lenovo_ssdt1.aml
iasl -d lenovo_ssdt2.aml          # use `-e thinkpad_dsdt.aml` to resolve externals
```

---

## SSDT1 — `TP-SSDT1` (trivial)

Single top-level `Scope (\) { Method (KOU1) { Stall (0x64) } }`.
`KOU1`/`KOU2` are 100 µs "keep OS update" stalls — no functional logic. SSDT1 is effectively an anchor/stub; all real content is in SSDT2.

---

## SSDT2 — `TP-SSDT2` (LCD backlight)

Adds brightness control methods to **both** display paths:

### 1. `\_SB.PCI0.VID.LCD0` (integrated GPU)
- **`_BCL`** (brightness levels): if `\WVIS` → `\NBCF=1`. If `\WIN8` returns a 103-slot package (`0x64,0x64,0x00..0x64`); else returns `EC.BRTW` (backlight table).
- **`_BCM(A)`** (set): Win8 path → `VID.AINT(0x01, EC.BRCD(A))`. Legacy path → `Match(BRTW, MEQ, A)`; store `\BRLV`, then `EC.BRNS(UCMS(0x16))`.
- **`_BQC`** (query): `BRLV+2` index into `EC.BRTW`.

### 2. `\_SB.PCI0.PEG.VID.LCD0` (discrete GPU variant)
- Same `_BCL` (drops the `BFRQ`/`PFMB` side-work).
- **`_BCM`**: Win7+ISOP path delegates to `VID.LCD0._BCM`; else unused on dGPU. Winner8/NBCF path uses `Match(BRTW)`, `\VBRC(BRLV)`.
- **`_BQC`**: queries integrated `_BQC` when `WIN7 && ISOP`.

> `\NBCF` = "new backlight control flow" flag toggled by the Intel `AINT`/`PFMB`/`BRSQ` path. `\BRLV` is the OS-side level; `EC.BRTW/BRCD/BRNS/BFRQ` are the EC-backed brightness registers (DSDT lines 11129–11189).

---

## HKEY device — `\_SB.PCI0.LPC.EC.HKEY` (`LEN0068`)

The classic ThinkPad "ACPI hotkey / user-interface event" device (same interface the Linux `thinkpad_acpi` driver consumes).

```asl
Device (HKEY)
{
    Name (_HID, EisaId ("LEN0068"))
    Name (DHKC, 0x00)   // hotkey subsystem enabled flag
    Name (DHKN, 0x0808) // 32-bit hotkey enable mask (default)
    Mutex (XDHK, 0x00)  // guard for all queue reads/writes
    ...DHKH/DHKW/DHKS/DHKD/DHKT/DHWW event queues...
}
```

### Method reference

| Method | Args | Purpose |
| ------ | ---- | ------- |
| `MHKV` | — | version, returns `0x0100` |
| `MHKA` | — | available-features mask, returns `0xFFFFFFFB` (all bits except bit 2) |
| `MHKN` | — | returns current hotkey mask `DHKN` |
| `MHKK` | 1 | status probe: `if (DHKC) return (DHKN & Arg0)` else `0` — tests whether a hotkey bit is enabled |
| `MHKM` | 2 | set/clear mask bits: `DHKN \|= / &= (1<<Arg0)` for Arg0 ≤ 0x20, skipping bits covered by `0xFFFFFFFB` |
| `MHKC` | 1 | `DHKC = Arg0` — enable/disable the whole hotkey path |
| `MHKE` | 1 | reset: clear all event queues, set `DHKB=Arg0` |
| `MHKQ` | 1 | **push event** `Arg0` into one queue by code range (below); then `Notify(HKEY, 0x80)`; special case `0x1004` → `Notify(\_SB.SLPB, 0x80)` |
| `MHKP` | — | **pop next queued event** in priority order: `DHWW→DHKW→DHKD→DHKS→DHKT→DHKH`, clearing it; returns the code |
| `MHKS` | — | `Notify(\_SB.SLPB, 0x80)` |
| `MHKB` | 1 | lid beep + `\LIDB` update (`BEEP(0x11)` close / `BEEP(0x10)` open) |
| `MHKD` | — | video: calls `VID.VLOC`/`PEG.VID.VLOC` (0x00) when `\PLUX==0` |
| `MHQC/MHGC/MHSC/CKC4/MHQE/MHGE/MHSE` | — | C4/on-demand-AC (`\CWAC`,`\CWAS`,`\C4AC`,`\C4WR`) state helpers |
| `MLCG/MLCS` | — | keyboard-backlight get/set via `\KBLS`; emits `MHKQ(0x6001)` / `0x1012` |
| `PBLG/PBLS` | — | power-brightness get/set (`\BRLV`); `PBLS` → `MHKQ(0x6050)` |
| `PMSG/PMSS` | — | power-mode get/set via `\PRSM` |
| `ISSG/ISSS` | — | input-device status (`\ISSP`,`\ISFS`,`\ISCG`) |
| `FFSG/FFSS` | — | fan/fullscreen state via `\IFRS` |
| `GMKS/SMKS` | — | Fn-key mode get/set (`FNSC(0x02)`, toggles `EC.HKLK` × `0x0200`), emits `MHKQ(0x6060)` |
| `INSG/INSS` | — | input (touchpad/trackpoint) enable/disable via `\IOEN`,`\IOST`,`\IOCP` |
| `DSSG/DSSS` | — | display-state get/set (`0x0400 \| \PDCI`) |
| `SBSG/SBSS` | — | S/B state via `\SYBC` |
| `MHGI/MHSI` | — | (MHGI sub-scope) info get/set: battery `\IPMB/IPMR/IPMO/IPMA` (arg 0,1,&,8,9), dynamic voltage `\VDYN` |
| `MHQI` | — | returns `0x00` |
| `MMTG/MMTS` | — | mute-mode get/set (`HDMC`), set `F4LD` |
| `PWMC/PWMG` | — | fan-PWM present check; `PWMG` = `(EC.PWMH<<8)|EC.PWML` |
| `UAWO` | — | `\UAWS(Arg0)` wake hook |

### MHKQ event queue classification

| Code range | Queue | Meaning (thinkpad_acpi convention) |
| ---------- | ----- | ---------------------------------- |
| `0x1000–0x1FFF` | `DHKH` | hotkeys (Fn-combos) |
| `0x2000–0x2FFF` | `DHKW` | wake sources |
| `0x3000–0x3FFF` | `DHKS` | suspend/docking |
| `0x4000–0x4FFF` | `DHKD` | dock events |
| `0x5000–0x5FFF` | `DHKH` | hotkey extension |
| `0x6000–0x6FFF` | `DHKT` | "toast" / status-change notifications |
| `0x7000–0x7FFF` | `DHWW` | wireless-wake / hibernate |
| else | ignored | `0x1004` short-circuits to `Notify(SLPB)` |

Consumption order (driver polling `MHKP`): wake → hotplug → dock → suspend → toast → hotkey.

---

### EC query → event dispatch (complete)

Each `_Qxx` is raised by the EC writing its query byte over LPC; the handler gates on `MHKK(bit)` before pushing the event. Bitmask bbelow is from `DHKN` (enabled-hotkey gate).

| EC query | Gate bit (`MHKK`) | Pushed (`MHKQ`) | Likely source |
| -------- | ----------------- | ---------------- | ------------- |
| `_Q10` | `0x01` | `0x1001` | Fn key A |
| `_Q12` | `0x02` | `0x1002` | Fn key B |
| `_Q13` | `DHKC` | `0x1004` | power button / sleep (→SLPB) |
| `_Q64` | `0x10` | `0x1005` | Fn key |
| `_Q65` | `0x20` | `0x1006` | Fn key |
| `_Q16` | `0x40` | `0x1007` | Fn key |
| `_Q17` | `0x80` | `0x1008` | Fn key |
| `_Q18` | `0x100` | `0x1009` | Fn key |
| `_Q1A` | `0x400` | `0x100B` | Fn key |
| `_Q1B` | — | `0x100C` | Fn key |
| `_Q62` | `0x1000` | `0x100D` | Fn key |
| `_Q60` | `0x2000` | `0x100E` | Fn key |
| `_Q61` | `0x4000` | `0x100F` | Fn key |
| `_Q14` | `0x8000` | `0x1010` + `0x6050` | Fn+F2 brightness |
| `_Q15` | `0x10000` | `0x1011` + `0x6050` | Fn+F3 brightness |
| `_Q63` | `0x80000` | `0x1014` | Fn+F6 |
| `_Q1F` | `0x20000` | `0x1012` | Fn key switch |
| `_Q19` | `0x800000` | `0x1018` | Fn+F8 |
| `_Q1C` | `0x1000000` | `0x1019` | Fn key |
| `_Q1D` | `0x2000000` | `0x101A` | Fn key |
| `_Q6A` | `0x4000000` | `0x101B` | Fn key |
| `_Q66` | `0x10000000` | `0x101D` | Fn key |
| `_Q67` | `0x20000000` | `0x101E` | Fn key |
| `_Q68` | `0x40000000` | `0x101F` | Fn key |
| `_Q69` | `0x80000000` | `0x1020` | Fn key |
| `_Q26`/`_Q27` | — | `0x6040` | AC/battery status toast |
| `_Q2A` | — | `0x60D0`, `0x5002` | dock attach |
| `_Q2B` | — | `0x60D0`, `0x5001` | dock detach |
| `_Q4E` | — | `0x6011` | battery-low frontend |
| `_Q4F`/`_Q46` | — | `0x6012` | battery critical |
| `_Q24` | `0x20000` | `0x6001`,`0x1012`,`0x6050`,`0x6060` | Fn-key state / keyboard-light |
| `_Q45` | — | `0x4010`,`0x4011` | dock(x4-style) port events |
| `_Q3F` | — | `0x6000` | hotkey-Fun |
| `_Q74` | — | `0x6060` | keyboard-light toggle |
| `_Q73` | — | `0x6005` | misc hotkey |
| `_Q38` | — | `0x3003` | suspend hint |
| `_Q41` | — | `0x7000`,`0x6070`,`0x6080`,`0x6030` | hibernate+wake |
| `_Q40` | — | `0x6022`,`0x6030` | wake source |

> Handlers without entries here (`_Q11,_Q3D,_Q48,_Q49,_Q7F,_Q22,_Q4A,_Q4B,_Q7B,_Q78,_Q90,_Q91,_Q2C,_Q2D,_Q43,_Q70,_Q72`) perform hotel-specific actions like fan sweeps, thermal-logic, or are legacy no-ops — verify against the `.dsl` before relying on them.

### Hotkey KeyMap (Fn) — best-match from ThinkPad convention

`0x1001–0x100C` → Fn+Function-row pairs; `0x100D–0x1020` → Fn combos for special actions. Final labeling (e.g., which Fn key maps to which function) should be confirmed against `drivers/platform/x86/thinkpad_acpi.c`'s hotkey table for this generation.

---

## How the pieces fit

```
EC raises query byte (LPC port 62h)
   → firmware DSDT _Qxx handler
       → HKEY.MHKK(bit)  == enabled? (gate on DHKN mask)
           → HKEY.MHKQ(code)  → queued by range + Notify(HKEY,0x80)
               → lenovo-acpi (OS driver) calls MHKP() to dequeue
                   → OS action (wifi toggle, backlight, ACPI sleep…)
```

## Files added / modified

- `lenovo_ssdt1.aml` / `lenovo_ssdt1.dsl` — carved + decompiled SSDT1
- `lenovo_ssdt2.aml` / `lenovo_ssdt2.dsl` — carved + decompiled SSDT2
- `guide.md` — "Might-TODO" items for SSDT + HKEY now complete