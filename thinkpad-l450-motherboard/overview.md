# Lenovo ThinkPad L450 Reverse Engineering

Board: AIVL1 NM-A351 REV 1.0
CPU: Intel i5-5300U
Board Date: 2014-11-10

---

## Phase 0 - Identification

- [x] Identify motherboard model
- [x] Identify CPU model
- [ ] Photograph front side
- [ ] Photograph back side
- [ ] Create motherboard repository
- [ ] Create project structure
- [ ] Record board dimensions
- [ ] Document all connectors

---

## Phase 1 - Component Inventory

### Major ICs

- [ ] Locate CPU
- [ ] Locate Platform Controller Hub (PCH)
- [ ] Locate Embedded Controller (EC)
- [ ] Locate BIOS flash chip
- [ ] Locate Ethernet controller
- [ ] Locate Audio codec
- [ ] Locate USB controllers
- [ ] Locate Power Management ICs

### Documentation

- [ ] Photograph each major IC
- [ ] Record part numbers
- [ ] Download datasheets
- [ ] Create chip inventory table
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
- [ ] Identify charger IC
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

- [ ] Locate BIOS flash chip
- [ ] Identify manufacturer
- [ ] Determine flash capacity
- [ ] Dump BIOS image

### Analysis

- [ ] Inspect UEFI structure
- [ ] Extract firmware modules
- [ ] Identify Intel ME region
- [ ] Document firmware layout

### Tools

- [ ] Configure flashrom
- [ ] Configure UEFITool
- [ ] Configure binwalk

---

## Phase 5 - Embedded Controller

### Identification

- [ ] Identify EC model
- [ ] Obtain datasheet
- [ ] Document pinout

### Function Analysis

- [ ] Keyboard interface
- [ ] Battery interface
- [ ] Fan control
- [ ] Power button handling
- [ ] Sleep state management

---

## Phase 6 - Bus Mapping

### SPI

- [ ] Trace EC ↔ BIOS
- [ ] Trace CPU/PCH ↔ BIOS

### SMBus / I²C

- [ ] Trace battery communications
- [ ] Trace sensor communications

### LPC / eSPI

- [ ] Trace EC ↔ PCH

### USB

- [ ] Trace USB root ports
- [ ] Trace internal USB devices

### Documentation

- [ ] Create bus diagrams

---

## Phase 7 - Boot Process

### Sequence Reconstruction

- [ ] Power button event
- [ ] EC startup
- [ ] Power rail enable sequence
- [ ] CPU reset release
- [ ] BIOS execution
- [ ] POST sequence
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
- [ ] Firmware notes
- [ ] Component database
- [ ] Publish findings