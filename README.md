# South African Railway Locomotive GPS Tracking System
**Cellular-Base with Modular Extensions + Train Management Integration**

## Overview

A comprehensive GPS tracking system for locomotive fleet management across the South African rail network. This solution provides real-time location data through cellular connectivity with modular expansion capabilities for LoRa and satellite communication, **plus complete train management integration** with existing Transnet systems (ITP, TMS, VDU) to solve both locomotive tracking and train identification challenges.

**Solution:** LILYGO T-SIM7600G-H platform with enhanced user interface, cellular-first approach, optional expansion modules, and direct integration with Transnet's train management ecosystem.

## Key Features

- **Complete Train Management** - Full integration with ITP, TMS, and VDU systems for train identification
- **Enhanced User Interface** - 2.8" color TFT display, 4-button navigation, train assignment workflow
- **Immediate Deployment** - Works out-of-box with cellular networks
- **Over-the-Air Updates** - Remote firmware deployment to entire fleet via cellular
- **Modular Expansion** - Add LoRa or satellite modules for dead zones  
- **Real-time Tracking** - 1-minute update intervals on main routes
- **Private Network Access** - APN configuration for secure company network
- **Fleet Management** - Centralized monitoring and control of all tracking units
- **Cost-Optimized** - R3,650 base unit with enhanced UI, optional modules as needed
- **TCO Workflow Integration** - Seamless train number assignment and confirmation process
- **Future-Proof** - Expandable design accommodates new technologies

## Quick Start

1. **Hardware**: LILYGO T-SIM7600G-H development board
2. **Network**: Configure private APN with cellular provider
3. **Database**: MS SQL Server with spatial extensions  
4. **Deploy**: Install on test locomotives for pilot validation

## Project Structure

```
locomotive-tracking-unit/
├── firmware/           # ESP32 embedded software
│   ├── core/          # Core tracking functionality
│   ├── modules/       # Communication module drivers
│   ├── config/        # Configuration management
│   └── tests/         # Unit tests
├── hardware/          # PCB designs and schematics
│   ├── base-unit/     # Main tracking unit design
│   ├── expansion-modules/  # Optional communication modules
│   ├── schematics/    # Circuit diagrams
│   └── bom/           # Bills of materials
├── backend/           # Server-side applications
│   ├── api/           # .NET Core Web API
│   ├── services/      # Background services
│   ├── database/      # Data access layer
│   └── mqtt/          # MQTT broker configuration
├── frontend/          # Web applications
│   ├── dashboard/     # Real-time monitoring dashboard
│   └── mobile/        # Mobile applications
├── database/          # Database schemas and migrations
│   ├── schemas/       # SQL Server table definitions
│   ├── migrations/    # Database version control
│   └── seed-data/     # Initial data setup
├── deployment/        # Installation and configuration
│   ├── installation/  # Hardware installation guides
│   ├── configuration/ # System configuration
│   └── monitoring/    # System monitoring setup
├── tools/             # Development utilities
│   ├── simulator/     # GPS and network simulators
│   ├── configurator/  # Configuration tools
│   └── testing/       # Testing utilities
├── tests/             # Testing frameworks
│   ├── unit/          # Unit test suites
│   ├── integration/   # Integration tests
│   └── field/         # Field testing procedures
├── docs/              # Documentation
│   ├── api/           # API documentation
│   ├── hardware/      # Hardware documentation
│   ├── deployment/    # Deployment guides
│   ├── user-guide/    # User manuals
│   └── concepts/      # Research and concept documents
└── .archive/          # Legacy files and historical reference
```

## Technology Stack

### Hardware
- **Core Platform**: LILYGO T-SIM7600G-H (ESP32 + 4G LTE + GPS)
- **Enhanced UI**: 2.8" color TFT display (320x240), 4-button navigation pad, multi-color status LEDs
- **Expansion**: Optional LoRa and satellite communication modules
- **Power**: 110V railway power with 48-hour battery backup (0.6W idle, 7.7W peak including UI)
- **Environment**: IP67 rated, -20°C to +70°C operation

### Software  
- **Embedded**: C++ on ESP32 using PlatformIO with OTA support and train management workflows
- **Backend**: .NET Core Web API with Entity Framework and train management integration services
- **Database**: MS SQL Server with spatial data extensions and train management tables
- **Frontend**: React.js dashboard with real-time mapping and fleet management
- **Communication**: MQTT over TLS for secure data transmission and train assignment commands
- **Integration**: Direct API connectivity with Transnet ITP, TMS, and VDU systems
- **OTA System**: Secure firmware updates with rollback protection

### Network
- **Primary**: Cellular (4G/3G/2G) via private APN
- **Optional**: LoRa 868MHz for depot areas
- **Optional**: Satellite (Swarm/Iridium) for remote routes

## Communication Strategy

| Module | Coverage | Monthly Cost | Use Case |
|--------|----------|--------------|----------|
| **Cellular (Base)** | 99%+ urban/main routes | R50 | Primary tracking |
| **LoRa (Optional)** | Depot areas, gateways | R0 | Cost optimization |
| **Satellite (Optional)** | 100% global | R75-500 | Remote routes |

## Getting Started

### Development Setup

```bash
# Clone repository
git clone https://github.com/your-org/locomotive-tracking-unit.git
cd locomotive-tracking-unit

# Setup ESP32 development environment with OTA support
pip install platformio
cd firmware/core
pio init --board esp32dev
# Configure OTA partitions in platformio.ini

# Setup backend development
cd ../../backend/api
dotnet restore
dotnet build

# Setup frontend development  
cd ../../frontend/dashboard
npm install
npm start
```

### OTA Update System

The system includes comprehensive Over-the-Air update capabilities:

- **Fleet-Wide Updates**: Deploy firmware to entire locomotive fleet remotely
- **Staged Rollouts**: Canary deployments with automatic rollback protection
- **Security**: Signed firmware with certificate-based authentication
- **Monitoring**: Real-time update progress and success tracking
- **Zero Downtime**: Updates happen during scheduled maintenance windows

### Hardware Setup

1. **Acquire Hardware**
   - LILYGO T-SIM7600G-H development board
   - External GPS/4G antenna
   - Power supply and enclosure

2. **Configure Network**
   - Obtain SIM card with data plan
   - Configure APN settings for private network access
   - Test cellular connectivity

3. **Deploy Firmware**
   - Flash firmware to ESP32
   - Configure device ID and network settings
   - Validate GPS and cellular functionality

## Cost Summary

### Initial Investment (100 Units)
- Base Hardware: R365,000 (100 × R3,650, includes enhanced UI)
- Development & Integration: R180,000 (includes train management integration)
- Installation: R50,000
- **Total CAPEX: R595,000**

### Monthly Operating Costs (100 Units)
- Cellular Connectivity: R4,250 (85 units)
- Satellite Data: R875 (5 units)  
- System Maintenance: R500
- **Total OPEX: R5,625/month**

## Implementation Timeline

- **Phase 1 (Months 1-6)**: Cellular deployment for immediate tracking
- **Phase 2 (Months 7-12)**: LoRa gateway infrastructure and module deployment  
- **Phase 3 (Months 13-18)**: Satellite modules for remote route coverage

## Documentation

- **[Implementation Guide](docs/implementation-guide.md)** - Complete technical documentation
- **[Hardware Specifications](hardware/README.md)** - Detailed component information  
- **[Executive Summary](docs/executive-summary.md)** - Project overview and business case
- **[Firmware Documentation](firmware/README.md)** - ESP32 firmware implementation
- **[Backend Documentation](backend/README.md)** - Server-side implementation

## Research & Concepts

This project evolved from comprehensive analysis of five communication concepts and hardware platform options. The complete research documentation provides insight into the decision-making process:

- **[Research & Design Concepts](docs/concepts/)** - Complete research documentation and analysis
- **[Concept Analysis](docs/concepts/concept-analysis.md)** - Five communication approaches evaluated
- **[Decision Rationale](docs/concepts/decision-rationale.md)** - Why cellular-base modular system was chosen
- **[Hardware Analysis](docs/concepts/hardware-analysis.md)** - ESP32 vs Raspberry Pi platform comparison

## Support

- **Technical Issues**: Create issue in this repository
- **Hardware Support**: Contact LILYGO or local distributors  
- **Network Support**: Contact cellular service provider

## License

Proprietary - Internal use only.

---

*Developed for South African railway operations with focus on practical deployment, cost-effectiveness, and operational reliability.*