# Lenovo ThinkPad L450 Reverse Engineering

Board: AIVL1 NM-A351 REV 1.0
CPU: Intel i5-5300U
Board Date: 2014-11-10

---

## Phase 0 - Identification
 
- [x] Identify motherboard model
- [x] Identify CPU model
- [x] Create motherboard repository
- [x] Create project structure
- [ ] Document all connectors

---

## Phase 1 - Component Inventory

### Major ICs

- [ ] Locate CPU
- [ ] Locate Platform Controller Hub (PCH)
- [x] Locate Embedded Controller (EC)
- [x] Locate BIOS flash chip
- [x] Locate Ethernet controller
- [x] Locate Audio codec
- [x] Locate USB controllers
- [x] Locate Power Management ICs

### Documentation

- [ ] Photograph each major IC
- [x] Record part numbers
- [ ] Download datasheets
- [x] Create chip inventory table
- [ ] Label motherboard map

---

## Phase 2 - Connector Analysis

### External Connectors

- [ ] USB ports
- [ ] HDMI/VGA
- [ ] Ethernet
- [ ] Audio jack
- [ ] Power input

### Internal Connectors

- [ ] Keyboard
- [ ] Touchpad
- [ ] LCD
- [ ] Battery
- [ ] Speakers
- [ ] Webcam
- [ ] WiFi card

### Documentation

- [ ] Trace connector destinations
- [ ] Create connector map

---

## Phase 3 - Power System

### Power Input

- [ ] Locate DC input path
- [ ] Identify protection circuitry
- [x] Identify charger IC
- [ ] Identify battery management circuitry

### Power Rails

- [ ] Identify 20V rail
- [ ] Identify 5V rail
- [ ] Identify 3.3V rail
- [ ] Identify CPU core rail
- [ ] Identify RAM rail
- [ ] Identify chipset rail

### Verification

- [ ] Measure continuity
- [ ] Trace power paths
- [ ] Build power tree diagram

---

## Phase 4 - Firmware Analysis

### BIOS

- [x] Locate BIOS flash chip
- [x] Identify manufacturer
- [x] Determine flash capacity
- [x] Dump BIOS image

### Analysis

- [x] Inspect UEFI structure
- [x] Extract firmware modules
- [~] Identify Intel ME region *ME region confirmed absent from FL1 capsule, need more analysis
- [x] Document firmware layout

### Tools

- [x] Configure flashrom
- [x] Configure UEFITool
- [x] Configure binwalk

---

## Phase 5 - Embedded Controller

### Identification

- [x] Identify EC model
- [x] Obtain datasheet
- [ ] Document pinout

### Function Analysis

- [ ] Keyboard interface
- [ ] Battery interface
- [x] Fan control
- [x] Power button handling
- [x] Sleep state management

---

## Phase 6 - Bus Mapping

### SPI

- [ ] Trace EC ↔ BIOS
- [ ] Trace CPU/PCH ↔ BIOS

### SMBus / I²C

- [ ] Trace battery communications
- [x] Trace sensor communications *(Mapped Thermal Sensors HT0/HT1 in DSDT)*

### LPC / eSPI

- [x] Trace EC ↔ PCH *(Mapped ECOR space over LPC bus)*

### USB

- [ ] Trace USB root ports
- [ ] Trace internal USB devices

### Documentation

- [ ] Create bus diagrams

---

## Phase 7 - Boot Process

### Sequence Reconstruction

- [x] Power button event *(Mapped HWAK tracking logic)*
- [x] EC startup *(Mapped ECOR fields and lifecycle initialization)*
- [ ] Power rail enable sequence
- [ ] CPU reset release
- [x] BIOS execution *(Isolated DxeMain core execution initialization)*
- [x] POST sequence *(Mapped physical Port 80 tracking loop via D80P)*
- [ ] Boot device initialization

### Documentation

- [ ] Create boot flowchart
- [ ] Create timing diagram

---

## Phase 8 - PCB Reverse Engineering

### Routing

- [ ] Trace CPU power phases
- [ ] Trace RAM routing
- [ ] Trace PCIe lanes
- [ ] Trace USB lanes

### PCB Structure

- [ ] Identify ground planes
- [ ] Identify power planes
- [ ] Identify differential pairs

---

## Phase 9 - Board Bring-Up

- [ ] Verify shorts
- [ ] Apply bench power
- [ ] Observe power rails
- [ ] Observe boot behavior
- [ ] Record POST status

---

## Phase 10 - Final Documentation

- [ ] Motherboard atlas
- [ ] Annotated board images
- [ ] Power tree diagram
- [ ] Bus map
- [ ] Boot process map
- [x] Firmware notes
- [ ] Component database
- [ ] Publish findings