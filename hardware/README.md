# Hardware

Hardware designs and specifications for the locomotive GPS tracking system.

## Overview

The hardware is built around the LILYGO T-SIM7600G-H platform, providing integrated ESP32 processing, 4G cellular communication, and GPS positioning in a single development board.

## Directory Structure

```
hardware/
├── base-unit/         # Main tracking unit design
│   ├── pcb/          # PCB layout files (KiCad)
│   ├── enclosure/    # 3D models and mounting hardware
│   ├── assembly/     # Assembly instructions and diagrams
│   └── testing/      # Hardware test procedures
├── expansion-modules/ # Optional communication modules
│   ├── lora-module/  # LoRa 868MHz module design
│   ├── satellite-module/  # Satellite communication module
│   └── interface/    # Module connector specifications
├── schematics/       # Circuit diagrams and designs
│   ├── power-supply/ # Railway power interface circuits
│   ├── protection/   # Surge and EMI protection
│   └── connectors/   # External connection diagrams
└── bom/              # Bills of materials and sourcing
    ├── base-unit-bom.csv    # Core unit components
    ├── suppliers.md         # Component supplier information
    └── cost-analysis.xlsx   # Cost breakdown and analysis
```

## Base Unit Specifications

### Core Components
- **Main Board**: LILYGO T-SIM7600G-H (ESP32 + 4G LTE + GPS)
- **Enhanced UI**: 2.8" color TFT display (320x240), 4-button navigation pad, multi-color status LEDs
- **GPIO Expander**: MCP23017 I2C 16-port expander (mandatory for enhanced UI)
- **Unit Price**: R3,675 (includes enhanced UI components + GPIO expander)
- **Dimensions**: 120mm × 90mm × 25mm (increased for UI integration)

### Power System
- **Input**: 110V DC railway power
- **Power Supply**: Mean Well DDR-30G-5 (5V/6A output) - upgraded for UI power requirements
- **Backup Battery**: 2× 18650 Li-ion (7.4V, 6800mAh)
- **Battery Life**: 38 hours continuous operation (with enhanced UI active)
- **Power Consumption**: 0.65W idle (including UI + GPIO expander), 7.7W peak transmission

### Environmental Protection
- **Enclosure Rating**: IP67 (dust/water resistant)
- **Operating Temperature**: -20°C to +70°C
- **Humidity**: 95% RH non-condensing
- **Vibration Resistance**: IEC 61373 Class 1
- **EMC Compliance**: EN 50121 railway standard

### Connectivity
- **Cellular Antenna**: 4G/GPS combo antenna (external)
- **WiFi**: Built-in ESP32 WiFi (2.4GHz) - used for debug mode Access Point
- **Expansion**: 4× M.2 Key-E slots for optional modules
- **Data Storage**: 32GB industrial MicroSD card
- **Enhanced UI Display**: 2.8" color TFT touch display (320x240 resolution)
- **User Interface**: 4-button navigation pad (Up/Down/Left/Right + OK/Cancel)
- **Status Indicators**: 4× multi-color LEDs (GPS, Cellular, Train Assignment, System Status) + 1× debug status LED

## Bill of Materials (Base Unit)

| Component | Model | Quantity | Unit Price (ZAR) | Total |
|-----------|--------|----------|------------------|-------|
| Main Board | LILYGO T-SIM7600G-H | 1 | R1,500 | R1,500 |
| TFT Display | 2.8" Color TFT 320x240 | 1 | R350 | R350 |
| Button Assembly | 4-button navigation pad | 1 | R80 | R80 |
| Status LEDs | Multi-color LEDs x4 | 1 | R50 | R50 |
| GPIO Expander | MCP23017 I2C 16-port | 1 | R25 | R25 |
| Wiring Harness | UI connection cables | 1 | R50 | R50 |
| External Antenna | 4G/GPS Combo Antenna | 1 | R200 | R200 |
| Power Supply | Mean Well DDR-30G-5 | 1 | R350 | R350 |
| Backup Battery | 18650 Li-ion x2 + BMS | 1 | R120 | R120 |
| Storage | 32GB Industrial SD | 1 | R150 | R150 |
| Enclosure | IP67 Railway Box (larger) | 1 | R450 | R450 |
| Expansion Board | Custom PCB with UI | 1 | R250 | R250 |
| Connectors | M12, terminal blocks | 1 | R100 | R100 |
| Protection | TVS diodes, filters | 1 | R100 | R100 |
| Assembly | Labor and testing | 1 | R100 | R100 |
| **TOTAL** | | | | **R3,675** |

## Expansion Modules

### LoRa Communication Module
- **Transceiver**: SX1262 (868MHz ISM band)
- **Range**: 15km line-of-sight
- **Power**: 100mW output
- **Cost**: R250

### Satellite Communication Module (Swarm)
- **Modem**: Swarm M138
- **Coverage**: Global
- **Latency**: 1-20 minutes
- **Cost**: R3,200

### Satellite Communication Module (Iridium)
- **Modem**: Iridium 9603N
- **Coverage**: Global real-time
- **Latency**: <30 seconds
- **Cost**: R4,850

## Installation Requirements

### Mechanical Mounting
- **Location**: Locomotive electrical cabinet
- **Mounting**: DIN rail or wall bracket
- **Access**: Front panel for enhanced UI display, buttons, and maintenance
- **UI Panel**: Front-accessible 2.8" display with navigation buttons
- **Ventilation**: Passive cooling with ventilation slots

### Electrical Connections
- **Power**: 110V DC locomotive power bus
- **Ground**: Locomotive chassis ground
- **Antenna**: Roof-mounted external antenna
- **Expansion**: M.2 slots for optional modules

### Environmental Considerations
- **Vibration Isolation**: Shock mounts for sensitive components
- **Thermal Management**: Heat dissipation for power supplies
- **EMI Shielding**: Filtered power input and shielded enclosure
- **Moisture Protection**: Sealed enclosure with desiccant

## Design Files

### PCB Design
- **Tool**: KiCad 6.0+
- **Layers**: 4-layer PCB for optimal signal integrity
- **Impedance Control**: 50Ω single-ended, 100Ω differential
- **Manufacturing**: Standard FR4, HASL finish

### Mechanical Design
- **Tool**: Fusion 360
- **Materials**: Aluminum enclosure, ABS plastic components
- **Manufacturing**: CNC machining for enclosures, 3D printing for prototypes

## Testing & Validation

### Hardware Tests
1. **Power Supply Test**: Validate input range and regulation
2. **Thermal Test**: Operating temperature range validation
3. **Vibration Test**: Railway environment simulation
4. **EMC Test**: Electromagnetic compatibility verification
5. **Environmental Test**: IP67 ingress protection validation

### Field Validation
- **Pilot Installation**: 10 units on test locomotives
- **Duration**: 6 months field trials
- **Conditions**: Various weather and operational scenarios
- **Metrics**: Reliability, performance, durability

## Over-the-Air (OTA) Updates

The hardware platform provides comprehensive OTA update capabilities enabling remote firmware deployment across the entire locomotive fleet without physical access to devices.

### OTA Hardware Architecture

#### Memory Layout for OTA
The ESP32 flash memory is partitioned to support dual-boot OTA updates:

```
┌─────────────────────────────────────────┐
│           ESP32 Flash Memory (4MB)       │
├─────────────────────────────────────────┤
│  Bootloader (0x1000) - 32KB             │
├─────────────────────────────────────────┤
│  Partition Table (0x8000) - 3KB         │
├─────────────────────────────────────────┤
│  OTA Data (0xD000) - 8KB                │
├─────────────────────────────────────────┤
│  App Partition 0 (0x10000) - 1.5MB      │
│  Current Firmware (Active)              │
├─────────────────────────────────────────┤
│  App Partition 1 (0x190000) - 1.5MB     │
│  OTA Target (Update Downloads Here)     │
├─────────────────────────────────────────┤
│  SPIFFS (0x310000) - 1MB                │
│  Configuration & Data Storage           │
└─────────────────────────────────────────┘
```

### Hardware Requirements for OTA

| Component | Requirement | T-SIM7600G-H Specification |
|-----------|-------------|----------------------------|
| **Flash Memory** | 4MB minimum | 4MB ✓ |
| **RAM** | 8MB PSRAM | 8MB PSRAM ✓ |
| **Cellular Modem** | 4G/3G/2G support | SIM7600G ✓ |
| **Power Supply** | Stable during updates | Railway + battery backup ✓ |
| **Watchdog Timer** | Hardware recovery | ESP32 built-in ✓ |

### OTA Network Infrastructure
- **Update Server**: Private network server (10.50.100.30)
- **MQTT Broker**: Fleet command distribution (10.50.100.10)
- **Private APN**: Secure cellular tunnel ("transnet.m2m")
- **HTTPS/TLS**: Encrypted firmware downloads
- **Certificate Auth**: Device identity verification

### Fleet Management Hardware
- **Status LED**: Visual update progress indication
- **OLED Display**: Local update status display
- **Reset Button**: Manual recovery if needed
- **Expansion Slots**: Future OTA-capable modules

### Power Considerations for OTA
- **Update Power Budget**: 2.5W during download phase
- **Battery Backup**: Maintains power during 45-60 second updates  
- **Power Monitoring**: Prevents updates on low battery (<50%)
- **UPS Integration**: Railway power with seamless backup switching

### Debug Mode Specifications
- **Antenna Requirements**: Existing ESP32 WiFi antenna adequate for short-range AP (10-50m range)

### Security Hardware Features
- **Secure Element**: Optional crypto accelerator
- **Hardware RNG**: True random number generation
- **Tamper Detection**: Physical security monitoring
- **Debug Interface**: JTAG disabled in production

## Certification Requirements

### South African Compliance
- **ICASA**: Type approval for cellular and LoRa modules
- **SABS**: Safety standards compliance
- **Railway Safety Regulator**: Railway equipment approval

### International Standards
- **IEC 61373**: Railway shock and vibration
- **EN 50121**: Railway EMC standard
- **IEC 60529**: IP protection rating
- **UN 38.3**: Battery transport safety

## Manufacturing & Assembly

### Production Volume
- **Pilot Run**: 25 units
- **Initial Production**: 100 units
- **Full Scale**: 500+ units annually

### Assembly Process
1. **PCB Assembly**: SMT placement and soldering
2. **Module Integration**: Install communication modules
3. **Enclosure Assembly**: Mechanical assembly and sealing
4. **Testing**: Functional and environmental tests
5. **Configuration**: Firmware programming and device setup

### Quality Control
- **Incoming Inspection**: Component verification
- **In-Process Testing**: Assembly stage validation  
- **Final Test**: Complete system functionality
- **Burn-in Test**: 48-hour operational validation

## Cost Analysis

### Unit Economics (100+ units)
- **Base Unit**: R3,675 per unit (includes enhanced UI + GPIO expander)
- **LoRa Module**: +R250 per unit (optional)
- **Satellite Module**: +R3,200-4,850 per unit (optional)
- **Installation**: R500 per unit

### Volume Pricing
- **25-99 units**: Standard pricing
- **100-499 units**: 10% discount
- **500+ units**: 15% discount + design optimization

---

*For technical specifications, see the [Implementation Guide](../docs/implementation-guide.md).*