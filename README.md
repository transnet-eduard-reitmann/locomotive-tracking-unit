# South African Railway Locomotive GPS Tracking System
**Cellular-Base with Modular Extensions**

## Overview

A comprehensive GPS tracking system for locomotive fleet management across the South African rail network. This solution provides real-time location data through cellular connectivity with modular expansion capabilities for LoRa and satellite communication.

**Solution:** LILYGO T-SIM7600G-H platform with cellular-first approach and optional expansion modules for comprehensive coverage.

## Key Features

- **Immediate Deployment** - Works out-of-box with cellular networks
- **Modular Expansion** - Add LoRa or satellite modules for dead zones  
- **Real-time Tracking** - 1-minute update intervals on main routes
- **Private Network Access** - APN configuration for secure company network
- **Cost-Optimized** - R3,120 base unit with optional modules as needed
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
- **Expansion**: Optional LoRa and satellite communication modules
- **Power**: 110V railway power with 48-hour battery backup
- **Environment**: IP67 rated, -20°C to +70°C operation

### Software  
- **Embedded**: C++ on ESP32 using PlatformIO
- **Backend**: .NET Core Web API with Entity Framework
- **Database**: MS SQL Server with spatial data extensions
- **Frontend**: React.js dashboard with real-time mapping
- **Communication**: MQTT over TLS for secure data transmission

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

# Setup ESP32 development environment
pip install platformio
cd firmware/core
pio init --board esp32dev

# Setup backend development
cd ../../backend/api
dotnet restore
dotnet build

# Setup frontend development  
cd ../../frontend/dashboard
npm install
npm start
```

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
- Base Hardware: R312,000 (100 × R3,120)
- Development & Integration: R150,000  
- Installation: R50,000
- **Total CAPEX: R512,000**

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
- **[Hardware Specifications](docs/hardware/)** - Detailed component information  
- **[API Documentation](docs/api/)** - Backend integration reference
- **[Deployment Guide](docs/deployment/)** - Installation procedures
- **[User Manual](docs/user-guide/)** - Operational documentation

## Research & Concepts

This project evolved from extensive research into multiple implementation approaches. Historical research documents and alternative implementations can be found in:

- **[Concept Research](docs/concepts/research/)** - Original 5-variant analysis
- **[Implementation Comparison](docs/concepts/research/implementation_comparison.md)** - Detailed variant comparison
- **[Original Overview](docs/concepts/research/original-overview.md)** - Historical project overview

## Support

- **Technical Issues**: Create issue in this repository
- **Hardware Support**: Contact LILYGO or local distributors  
- **Network Support**: Contact cellular service provider

## License

Proprietary - Internal use only.

---

*Developed for South African railway operations with focus on practical deployment, cost-effectiveness, and operational reliability.*