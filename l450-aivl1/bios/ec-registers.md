# IT8586E EC Register Map — AIVL1 / L450 (R06ET69W)

Definitive, offsets reconstructed exhaustively from `Field (ECOR ...)` blocks in
`thinkpad_dsdt.dsl` (EmbeddedControl region 0x00-0xFF, 7 Field blocks). 134 named bits/fields.

Format: `0xNN.BB` = byte offset.nn, bit index. Single bits are PS/2-style command values read at
`\_SB.PCI0.LPC.EC`. Word fields `SBB*`/`SB*`/`HWAC`/`HWAK`/`HSPD`/`HDEN`... are premium
SMBus/status reads.

| HDBM  | 0x00.0 (byte   0) | 1 | 0x00.0 |
| SBRC  | 0x00.0 (byte   0) | 16 | 0x00.0-15 |
| SBBM  | 0x00.0 (byte   0) | 16 | 0x00.0-15 |
| SBDC  | 0x00.0 (byte   0) | 16 | 0x00.0-15 |
| HKLK  | 0x00.2 (byte   0) | 1 | 0x00.2 |
| HFNE  | 0x00.3 (byte   0) | 1 | 0x00.3 |
| HLDM  | 0x00.6 (byte   0) | 1 | 0x00.6 |
| BBLS  | 0x00.7 (byte   0) | 1 | 0x00.7 |
| BTCM  | 0x01.0 (byte   1) | 1 | 0x01.0 |
| HBPR  | 0x01.4 (byte   1) | 1 | 0x01.4 |
| BTPC  | 0x01.5 (byte   1) | 1 | 0x01.5 |
| HDUE  | 0x01.6 (byte   1) | 1 | 0x01.6 |
| SBFC  | 0x02.0 (byte   2) | 16 | 0x02.0-15 |
| SBMD  | 0x02.0 (byte   2) | 16 | 0x02.0-15 |
| SBDV  | 0x02.0 (byte   2) | 16 | 0x02.0-15 |
| SNLK  | 0x02.3 (byte   2) | 1 | 0x02.3 |
| HAUM  | 0x03.1 (byte   3) | 2 | 0x03.1-2 |
| HSPA  | 0x03.3 (byte   3) | 1 | 0x03.3 |
| HSUN  | 0x03.4 (byte   3) | 8 | 0x03.4-11 |
| SBAE  | 0x04.0 (byte   4) | 16 | 0x04.0-15 |
| SBOM  | 0x04.0 (byte   4) | 16 | 0x04.0-15 |
| HSRP  | 0x04.4 (byte   4) | 8 | 0x04.4-11 |
| SBRS  | 0x06.0 (byte   6) | 16 | 0x06.0-15 |
| SBSI  | 0x06.0 (byte   6) | 16 | 0x06.0-15 |
| FNKS  | 0x06.2 (byte   6) | 1 | 0x06.2 |
| HLCL  | 0x06.3 (byte   6) | 8 | 0x06.3-10 |
| CALM  | 0x07.7 (byte   7) | 1 | 0x07.7 |
| HFNS  | 0x08.0 (byte   8) | 2 | 0x08.0-1 |
| SBAC  | 0x08.0 (byte   8) | 16 | 0x08.0-15 |
| SBDT  | 0x08.0 (byte   8) | 16 | 0x08.0-15 |
| NULS  | 0x09.0 (byte   9) | 1 | 0x09.0 |
| HAM0  | 0x09.1 (byte   9) | 8 | 0x09.1-8 |
| SBVO  | 0x0A.0 (byte  10) | 16 | 0x0A.0-15 |
| HAM1  | 0x0A.1 (byte  10) | 8 | 0x0A.1-8 |
| HAM2  | 0x0B.1 (byte  11) | 8 | 0x0B.1-8 |
| SBAF  | 0x0C.0 (byte  12) | 16 | 0x0C.0-15 |
| HAM3  | 0x0C.1 (byte  12) | 8 | 0x0C.1-8 |
| HAM4  | 0x0D.1 (byte  13) | 8 | 0x0D.1-8 |
| HAM5  | 0x0E.1 (byte  14) | 8 | 0x0E.1-8 |
| HAM6  | 0x0F.1 (byte  15) | 8 | 0x0F.1-8 |
| HAM7  | 0x10.1 (byte  16) | 8 | 0x10.1-8 |
| HAM8  | 0x11.1 (byte  17) | 8 | 0x11.1-8 |
| HAM9  | 0x12.1 (byte  18) | 8 | 0x12.1-8 |
| HAMA  | 0x13.1 (byte  19) | 8 | 0x13.1-8 |
| HAMB  | 0x14.1 (byte  20) | 8 | 0x14.1-8 |
| HAMC  | 0x15.1 (byte  21) | 8 | 0x15.1-8 |
| HAMD  | 0x16.1 (byte  22) | 8 | 0x16.1-8 |
| HAME  | 0x17.1 (byte  23) | 8 | 0x17.1-8 |
| HAMF  | 0x18.1 (byte  24) | 8 | 0x18.1-8 |
| HANT  | 0x19.1 (byte  25) | 8 | 0x19.1-8 |
| HANA  | 0x1A.3 (byte  26) | 2 | 0x1A.3-4 |
| SKEM  | 0x1A.6 (byte  26) | 1 | 0x1A.6 |
| HATR  | 0x1A.7 (byte  26) | 8 | 0x1A.7-14 |
| HT0H  | 0x1B.7 (byte  27) | 8 | 0x1B.7-14 |
| HT0L  | 0x1C.7 (byte  28) | 8 | 0x1C.7-14 |
| HT1H  | 0x1D.7 (byte  29) | 8 | 0x1D.7-14 |
| HT1L  | 0x1E.7 (byte  30) | 8 | 0x1E.7-14 |
| HFSP  | 0x1F.7 (byte  31) | 8 | 0x1F.7-14 |
| HMUT  | 0x21.5 (byte  33) | 1 | 0x21.5 |
| HUWB  | 0x22.0 (byte  34) | 1 | 0x22.0 |
| HWPM  | 0x22.1 (byte  34) | 1 | 0x22.1 |
| HWLB  | 0x22.2 (byte  34) | 1 | 0x22.2 |
| HWLO  | 0x22.3 (byte  34) | 1 | 0x22.3 |
| HWDK  | 0x22.4 (byte  34) | 1 | 0x22.4 |
| HWFN  | 0x22.5 (byte  34) | 1 | 0x22.5 |
| HWBT  | 0x22.6 (byte  34) | 1 | 0x22.6 |
| HWRI  | 0x22.7 (byte  34) | 1 | 0x22.7 |
| HWBU  | 0x23.0 (byte  35) | 1 | 0x23.0 |
| HWLU  | 0x23.1 (byte  35) | 1 | 0x23.1 |
| HWWL  | 0x23.2 (byte  35) | 1 | 0x23.2 |
| PIBS  | 0x23.6 (byte  35) | 1 | 0x23.6 |
| HPLO  | 0x24.2 (byte  36) | 1 | 0x24.2 |
| HWAC  | 0x24.3 (byte  36) | 16 | 0x24.3-18 |
| HB0S  | 0x26.3 (byte  38) | 7 | 0x26.3-9 |
| HB0A  | 0x27.2 (byte  39) | 1 | 0x27.2 |
| HB1S  | 0x27.3 (byte  39) | 7 | 0x27.3-9 |
| HB1A  | 0x28.2 (byte  40) | 1 | 0x28.2 |
| HCMU  | 0x28.3 (byte  40) | 1 | 0x28.3 |
| OVRQ  | 0x28.6 (byte  40) | 1 | 0x28.6 |
| DCBD  | 0x28.7 (byte  40) | 1 | 0x28.7 |
| DCWL  | 0x29.0 (byte  41) | 1 | 0x29.0 |
| DCWW  | 0x29.1 (byte  41) | 1 | 0x29.1 |
| HB1I  | 0x29.2 (byte  41) | 1 | 0x29.2 |
| KBLT  | 0x29.4 (byte  41) | 1 | 0x29.4 |
| BTPW  | 0x29.5 (byte  41) | 1 | 0x29.5 |
| FNKC  | 0x29.6 (byte  41) | 1 | 0x29.6 |
| HUBS  | 0x29.7 (byte  41) | 1 | 0x29.7 |
| BDPW  | 0x2A.0 (byte  42) | 1 | 0x2A.0 |
| BDDT  | 0x2A.1 (byte  42) | 1 | 0x2A.1 |
| HUBB  | 0x2A.2 (byte  42) | 1 | 0x2A.2 |
| BTWK  | 0x2A.4 (byte  42) | 1 | 0x2A.4 |
| HPLD  | 0x2A.5 (byte  42) | 1 | 0x2A.5 |
| HPAC  | 0x2A.7 (byte  42) | 1 | 0x2A.7 |
| BTST  | 0x2B.0 (byte  43) | 1 | 0x2B.0 |
| HPBU  | 0x2B.1 (byte  43) | 1 | 0x2B.1 |
| HBID  | 0x2B.3 (byte  43) | 1 | 0x2B.3 |
| HBCS  | 0x2B.7 (byte  43) | 1 | 0x2B.7 |
| HPNF  | 0x2C.0 (byte  44) | 1 | 0x2C.0 |
| GSTS  | 0x2C.2 (byte  44) | 1 | 0x2C.2 |
| HLBU  | 0x2C.5 (byte  44) | 1 | 0x2C.5 |
| DOCD  | 0x2C.6 (byte  44) | 1 | 0x2C.6 |
| HCBL  | 0x2C.7 (byte  44) | 1 | 0x2C.7 |
| SLUL  | 0x2D.0 (byte  45) | 1 | 0x2D.0 |
| HTMH  | 0x2D.1 (byte  45) | 8 | 0x2D.1-8 |
| HTML  | 0x2E.1 (byte  46) | 8 | 0x2E.1-8 |
| HWAK  | 0x2F.1 (byte  47) | 16 | 0x2F.1-16 |
| HMPR  | 0x31.1 (byte  49) | 8 | 0x31.1-8 |
| HMDN  | 0x33.0 (byte  51) | 1 | 0x33.0 |
| TMP0  | 0x33.1 (byte  51) | 8 | 0x33.1-8 |
| HIID  | 0x34.1 (byte  52) | 8 | 0x34.1-8 |
| HFNI  | 0x35.1 (byte  53) | 8 | 0x35.1-8 |
| HSPD  | 0x36.1 (byte  54) | 16 | 0x36.1-16 |
| TSL0  | 0x38.1 (byte  56) | 7 | 0x38.1-7 |
| TSR0  | 0x39.0 (byte  57) | 1 | 0x39.0 |
| TSL1  | 0x39.1 (byte  57) | 7 | 0x39.1-7 |
| TSR1  | 0x3A.0 (byte  58) | 1 | 0x3A.0 |
| TSL2  | 0x3A.1 (byte  58) | 7 | 0x3A.1-7 |
| TSR2  | 0x3B.0 (byte  59) | 1 | 0x3B.0 |
| TSL3  | 0x3B.1 (byte  59) | 7 | 0x3B.1-7 |
| TSR3  | 0x3C.0 (byte  60) | 1 | 0x3C.0 |
| HDAA  | 0x3C.1 (byte  60) | 3 | 0x3C.1-3 |
| HDAB  | 0x3C.4 (byte  60) | 3 | 0x3C.4-6 |
| HDAC  | 0x3C.7 (byte  60) | 2 | 0x3C.7-8 |
| RCBD  | 0x3D.1 (byte  61) | 1 | 0x3D.1 |
| RCWL  | 0x3D.2 (byte  61) | 1 | 0x3D.2 |
| RCWW  | 0x3D.3 (byte  61) | 1 | 0x3D.3 |
| HDEN  | 0x3D.4 (byte  61) | 32 | 0x3D.4-35 |
| HDEP  | 0x41.4 (byte  65) | 32 | 0x41.4-35 |
| HDEM  | 0x45.4 (byte  69) | 8 | 0x45.4-11 |
| HDES  | 0x46.4 (byte  70) | 8 | 0x46.4-11 |
| ATMX  | 0x47.4 (byte  71) | 8 | 0x47.4-11 |
| HWAT  | 0x48.4 (byte  72) | 8 | 0x48.4-11 |
| PWMH  | 0x49.4 (byte  73) | 8 | 0x49.4-11 |
| PWML  | 0x4A.4 (byte  74) | 8 | 0x4A.4-11 |

## Notes / groups

- **0x00-0x07 — system/general flags**: `HDBM` dock-pu, `HKLK` KB-Lock on dc, `HFNE` fan error,
  `HLDM` lid, `BBLS` big-Battery low-state, `BTCM`/`HBPR`/`BTPC` battery, `HDUE`,
  `SNLK` numeric-lock, `HAUM` audio-mute, `HSPA` sleep-wake, `HSUN`/`HSRP` email/RF? (8-bit bullets).
- **0x09-0x19 — HAM0..HAMF + HANT**: 16-byte "manufacturing" mailbox (thermal/aging), HANA 2-bit.
- **0x1B-0x1F — temperatures**: `HT0H/HT0L` CPU, `HT1H/HT1L` second, `HFSP` fan speed dith-string.
- **0x22-0x2A — hardware enable/status (`HW*`)**: webcam `HWPM`, lid `HWLB`, WLAN-off `HWLO`,
  dock `HWDK`, Fn `HWFN`, BT `HWBT`, RI `HWRI`, bay `HWBU`, `HWLU`, WLAN `HWWL`, dock-detect `HPLO`,
  `HWAC` 16-bit AC-present, battery `HB0S/HB1S` (7-bit sel + 1-bit action), `HCMU`, `OVRQ` over-read,
  `DCBD`, `DCWL/DCWW` dock-laptop-WL flag, `HB1I`, `KBLT` light, `BTPW` battery-power,
  `FNKC` Fn-key-combo, `HUBS` USB-hub power (see PUBS PowerResource), `BDPW` bay-power,
  `BDDT` bay-dock-state.
- **0x2B-0x33 — main battery/caps + hash**: `BTST`, `HPBU` p-battery-USB?, `HBID`, `HBCS`,
  `HPNF` panel-fault, `GSTS`, `HLBU`, `DOCD`, `HCBL`, `SLUL`, `HTMH`/`HTML` too-busy? 8-bit,
  `HWAK` 16-bit wake-cap count, `HMPR`, `HMDN`, `TMP0` ambient temperature (dedicated).
- **0x34-0x3B — EC index/ID/speed**: `HIID`, `HFNI` Fn-index, `HSPD` sleep-didle 16-bit,
  `TSL0..TSL3` temperature-sensor low 7-bit + `TSR0..3` status, `HDAA/HDAB/HDAC` thermal-ch all?
- **0x3F-0x4A — EC chipset support**: `RCBD/R CWL/RCWW` reset-config-WL flags, battery `HDEN/HDEP/HDEM/HDES`
  32/32/8/8-bit, `ATMX`, `HWAT`, `PWMH/PWML` fan-PWM 8-bit L/H.

> Cross-reference: same `H*` names appear in `_Qxx` EC-query handlers (acpi-hotkey.md) and in
> `PUBS._ON/_OFF` (USB power) — hardware people: correlate these with IT8586E pins on the board.

Reconstructed by: ASL Field-parser over decompiled DSDT (unpacked `_thinkpad_bios.bin.extracted`).
