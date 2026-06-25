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

Analysis:

* 7 pins each side (4 sides, square)
* 28 pin package
* not qfn
* from persepective: above the white stripe - down side pin 1 (black probe) connected to up side pin 5 (red probe) shows ".463v" on multimeter. upper pin 3, 7 also show values. pin 7 = 1.423v, pin 5 = 0.468v (all in continuity mode)
* Such values can be seen on each side of the package, but only if the black probe is on the lower side's pin 1.
* no beeps in any case

Verdict: Probably an active IC.

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


Verdict:

TPM/Security IC

## WINBOND 25064FVS10 1440

BIOS flash chip, 8MB of SPI flash which holds the UEFI/BIOS firmware.

## 472 APΔ .W49C

Unknown

## RAO5 AHΔ T51C

Unknown

## ALC3231 EBK2...(unreadable chars)

Unknown

## IT8556E 1451-FXS SC2TKA

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

Unknown

## 330 DEPK

Unknown

## Intel Dual Band Wireless-AC 7265

Wi-Fi 5 and bluetooth 4.2 network adapter device