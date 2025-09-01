# South African Railway Locomotive GPS Tracking System

## Overview

A comprehensive GPS tracking system for locomotive fleet management across the South African rail network, providing real-time location data through cellular connectivity with modular expansion capabilities for LoRa and satellite communication.

## Final Solution: Cellular-Base with Modular Extensions

After extensive research and evaluation of multiple implementation variants, the selected solution uses the **LILYGO T-SIM7600G-H** as the base platform, providing immediate cellular connectivity with the flexibility to add communication modules as needed.

### Key Features
- **Immediate Deployment** - Works out-of-box with cellular networks
- **Modular Expansion** - Add LoRa or satellite modules for dead zones
- **Real-time Tracking** - 1-minute update intervals on main routes
- **Private Network Access** - APN configuration for secure company network
- **Cost-Optimized** - R3,120 base unit with optional modules
- **Future-Proof** - Expandable as infrastructure develops

### Why This Solution?
- ✅ **Proven cellular technology** for immediate real-time tracking
- ✅ **No infrastructure required** to begin deployment
- ✅ **Modular design** allows adding LoRa when gateways are built
- ✅ **Satellite capability** can be added for remote routes
- ✅ **Single platform** simplifies maintenance and training
- ✅ **Cost-effective** compared to full modular system

---

## Technical Specifications

### Base Hardware
- **Platform:** LILYGO T-SIM7600G-H
- **Processor:** ESP32-WROVER (240MHz dual-core)
- **Cellular:** SIM7600G (4G LTE Cat-4, 3G, 2G fallback)
- **GPS:** Integrated GNSS (GPS, GLONASS, BeiDou, Galileo)
- **Storage:** 32GB MicroSD for data logging
- **Power:** 110V locomotive power with 48-hour battery backup
- **Operating Range:** -20°C to +70°C
- **Enclosure:** IP67 rated, vibration resistant

### Communication Options
| Module | Coverage | Cost | Use Case |
|--------|----------|------|----------|
| **Cellular (Base)** | 99%+ urban/main routes | R50/month | Primary tracking |
| **LoRa (Optional)** | Depot areas, gateways | R0/month | Cost optimization |
| **Satellite (Optional)** | 100% global | R75-500/month | Remote routes |

---

## System Architecture

```
┌─────────────────────────────────────┐
│   LOCOMOTIVE TRACKING UNIT          │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   T-SIM7600G-H BASE BOARD    │  │
│  │  • ESP32 Processor           │  │
│  │  • SIM7600G Cellular Modem   │  │
│  │  • Integrated GPS            │  │
│  │  • SD Card Storage           │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │   EXPANSION INTERFACE        │  │
│  │  • UART for Satellite        │  │
│  │  • SPI for LoRa              │  │
│  │  • I2C for Sensors           │  │
│  └──────────────────────────────┘  │
└─────────────────────────────────────┘
                    ↓
        [Private APN Connection]
                    ↓
    ┌──────────────────────────────┐
    │   COMPANY NETWORK             │
    │  • MS SQL Server Database     │
    │  • Fleet Management Dashboard │
    │  • Real-time Monitoring       │
    └──────────────────────────────┘
```

---

## Implementation Roadmap

### Phase 1: Base Deployment (Months 1-6)
- Deploy cellular-only units for immediate tracking
- 100 units at R3,120 each
- Real-time visibility across main routes

### Phase 2: LoRa Extension (Months 7-12)
- Build gateway infrastructure at depots
- Add LoRa modules to depot locomotives
- Reduce operational costs in coverage areas

### Phase 3: Satellite Extension (Months 13-18)
- Add satellite modules for remote routes
- Complete coverage including cross-border
- Full fleet optimization

---

## Cost Summary

### Initial Investment (100 Units)
| Component | Cost |
|-----------|------|
| Base Hardware (100x R3,120) | R312,000 |
| Development & Integration | R150,000 |
| Installation | R50,000 |
| **Total CAPEX** | **R512,000** |

### Operational Costs
| Configuration | Units | Monthly Cost |
|---------------|-------|--------------|
| Cellular Only | 85 | R4,250 |
| Cellular + LoRa | 10 | R500 |
| Cellular + Satellite | 5 | R875 |
| **Total Monthly** | **100** | **R5,625** |

---

## Variants Considered

During the research phase, five implementation variants were evaluated:

1. **Cellular-Only** - Simple but limited to cellular coverage
2. **LoRa-Only** - Low cost but no real-time capability
3. **Fixed Hybrid** - Expensive with redundant hardware
4. **Satellite-Only** - Excellent coverage but prohibitive costs
5. **Fully Modular** - Maximum flexibility but complex development

The selected solution combines the best aspects of these variants, using cellular as the proven base technology while maintaining the flexibility to add modules where needed.

---

## Key Benefits

### Immediate Value
- **Real-time tracking** from day one
- **Proven technology** with cellular networks
- **Quick deployment** without infrastructure development
- **Secure connectivity** via private APN

### Future Flexibility
- **Add modules as needed** based on operational data
- **Optimize costs** by using appropriate technology per route
- **Scale gradually** as budget allows
- **Future-proof** design accommodates new technologies

### Operational Excellence
- **99%+ coverage** on main routes with cellular
- **Zero dead zones** with optional modules
- **Unified platform** simplifies maintenance
- **Single dashboard** for all tracking data

---

## Documentation

- **[Implementation Guide](implementation_guide.md)** - Complete technical documentation
- **[Hardware Specifications](docs/hardware_specs.md)** - Detailed component information
- **[Software Architecture](docs/software_architecture.md)** - System design and protocols
- **[Installation Manual](docs/installation.md)** - Step-by-step deployment guide
- **[API Documentation](docs/api.md)** - Backend integration reference

---

## Getting Started

### Quick Start
1. **Order hardware:** LILYGO T-SIM7600G-H development kits
2. **Configure APN:** Work with network provider for private APN
3. **Setup database:** Configure MS SQL Server with spatial extensions
4. **Deploy pilot:** Test with 5-10 units on key routes
5. **Scale deployment:** Roll out based on pilot results

### Development Setup
```bash
# Clone repository
git clone https://github.com/transnet/locomotive-tracking.git
cd locomotive-tracking

# Setup ESP32 environment
pip install platformio
pio init --board esp32dev

# Build firmware
cd firmware
pio run

# Configure for private APN
# Edit config/apn_settings.h with your network details
```

---

## Support

- **Technical Issues:** railway-it@transnet.co.za
- **Hardware Support:** Contact LILYGO or local distributors
- **Network Support:** MTN Business / Vodacom Enterprise

---

## License

Proprietary - Transnet SOC Ltd. Internal use only.

---

*Developed for South African railway operations with focus on practical deployment, cost-effectiveness, and operational reliability.*