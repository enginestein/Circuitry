# Chip Inventory

## IT8586E

This is an input / output controller. Power management, keyboard inputs and system monitoring are the kind of tasks it handles.

Power button sends the first signal to this.

## NCP81101

This is a synchronous single phase buck controller. It's a step down controller, as it regulates the DC input to the required, lower DC value before passing it down the motherboard.

## NA69 LF

1000 base-T gigabit ethernet LAN magnetic filter

From what I know, ethernet cables, due to their architecture, generate a lot of noise and this device provides galvanic isolation which filters out all that electrical noise.

## 34AL 2503 V68LM 

Analysis

* 4 pins each side, a 16 pin package


## (+) 220 J40

This must be a capacitor, very likely placed above a fingerprint reader (FPR). the (+) sign indicates a polarized capacitor while the "220" suggests it's 220 microfarad. it is marked as C51 on the motherboard, most likely the capacitor C51.

## NEY jJ8

Most likely a film capacitor (from it's looks).

I found out that there's two metal components above both NEY jJ8 components, which is most likely an inductor which suggests that it might be a output capacitor for a buck converter (PC73, PC75), used for power rail filtering and satbilization (PJ2, PJ3)

There's a MOSFET nearby which says PQ22.

## 80 24780 I3 48 C3NZ

* 28-pin package (7 per side), manages the 20V DC-in path, battery/wall switching, charging buck.
* Multimeter observations consistent with a charge controller: internal resistance seen from ~1 pin (0.468V), higher diode drops on other pins — typical of the buck gate driver / protection FETs inside the package.

Verdict: **TI BQ24780S** (confirmed)

## L08-1 4496 

Analysis:

* 4 pins each two sides, 8 pin package

## P24JPVSP 7B 343 VG450

Analysis:

Continuity beep map:

pin 4 -> 4
pin 5 -> 10
pin 11 -> 4

total 14 pins each two sides.

UTPM1 is it's marking on the board, it might be a trusted platform module (specialized cryptographic chip designed to secure hardware)

Status: **Confirmed TPM/Security chip** (UTPM1 ref des, 3-pin beeps pattern matches LPC/SPI+direction strapping pins). L450-era ThinkPads ship Infineon SLB9660 (TPM 1.2) or SLB9665 — Vinafix board thread calls the L450 security chip "PS2408". To pin the exact model: probe VCC to GND (~3.3V daisy/standby rail) and confirm the serial pins run to the PCH.

Verdict:

**TPM/Security IC** (confirmed — exact vendor/model pending)

## WINBOND 25064FVS10 1440

BIOS flash chip, 8MB of SPI flash which holds the UEFI/BIOS firmware.

## 472 APΔ .W49C

Unknown

## RAO5 AHΔ T51C

Unknown

## ALC3231 EBK2...(unreadable chars)

Unknown

## IT8586E 1451-FXS SC2TKA

Unknown

## GL850G KJE0631 4491211

USB 2.0 hub controller, it splits a single USB upstream port from an intel CPU to multiple external USB ports.

## PI3L 720ZHE 1503GG

gigabit ethernet mux/demux. Simple mux logic but for ethernet data, which converts ethernet data from a single PHY device to two different ports without signal degradation.

## 3H 1M 22T

Unknown

## PS8338B A1 U08FAC AWMSP 0315

DisplayPort dual mode source demux (1:2), takes one DisplayPort (the device generating the video and audio signals) and routes it to two selectable DisplayPort outputs.

## 3257A 03 25 +8D450

Unknown

## 12 EAR39F43 G1218V

Unknown

## S472 ALΔ W51C

Unknown

## S412 FTΔ N49K

Unknown

## RH4VC

* Package: ~5–6 pin small SOT (2 pins one side, ~4 on other → SOT-23-5 or SOT-563/666)
* Pin-pin continuity: **two pins beep** (hard-shorted inside package)
* Pin-pin diode mode: **four pins read ~1.7V, no beep** (multi-junction: dual/Darlington transistor, 1.7V zener, or ESD rail through series element)
* Only continuity mode responds - resistance/voltage modes show OL/0. → junctions need >~1V test voltage (continuity mode sources higher V than ohms mode). Candidate list now includes red-LED-type junction (~1.7V).
* Not a plain power MOSFET (those read 0.5–0.7V)


## 330 DEPK

Unknown

## Intel Dual Band Wireless-AC 7265

Wi-Fi 5 and bluetooth 4.2 network adapter device