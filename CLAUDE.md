# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a South African Railway Locomotive GPS Tracking System - a comprehensive cellular-first modular tracking solution built around the LILYGO T-SIM7600G-H platform. The system provides real-time locomotive tracking with cellular connectivity, optional LoRa and satellite expansion modules, and over-the-air (OTA) update capabilities.

## Architecture

The system uses a **cellular-base with modular extensions** approach:

- **Hardware**: LILYGO T-SIM7600G-H (ESP32 + 4G LTE + GPS) as base unit with optional expansion modules
- **Firmware**: C++ on ESP32 using PlatformIO with comprehensive OTA support
- **Backend**: .NET Core Web API with Entity Framework, MQTT services, and SignalR
- **Database**: MS SQL Server with spatial data extensions  
- **Frontend**: React.js dashboard with real-time mapping and fleet management
- **Communication**: MQTT over TLS for secure data transmission

## Directory Structure

```
locomotive-tracking-unit/
├── firmware/           # ESP32 embedded software (C++/PlatformIO)
├── backend/           # .NET Core API and services
├── frontend/          # React.js dashboard and mobile apps
├── hardware/          # PCB designs and schematics
├── database/          # SQL Server schemas and migrations
├── deployment/        # Installation and configuration
├── tools/             # Development utilities and simulators
├── tests/             # Testing frameworks and procedures
├── docs/              # Documentation
└── .archive/          # Legacy files and historical reference
```

## Development Commands

### Firmware Development
```bash
# Setup ESP32 development environment
pip install platformio
cd firmware/core
pio init --board esp32dev

# Install required libraries
pio lib install "SIM7600"
pio lib install "TinyGPSPlus"  
pio lib install "ArduinoJson"
pio lib install "PubSubClient"

# Build firmware
pio run

# Flash to device
pio run --target upload --upload-port /dev/ttyUSB0

# Run tests
pio test
```

### Backend Development
```bash
# Setup .NET development
cd backend/api
dotnet restore
dotnet build

# Database setup
dotnet ef database update
dotnet run -- --seed-data

# Run API server
dotnet run

# Run background services
cd ../services
dotnet run

# Run tests
dotnet test
```

### Frontend Development
```bash
# Setup React dashboard
cd frontend/dashboard
npm install
npm start

# Build for production
npm run build

# Run tests
npm test
```

### System Services
```bash
# Start MQTT broker (Docker)
docker run -it -p 1883:1883 -p 9001:9001 eclipse-mosquitto

# Database with spatial support
sqlcmd -Q "ALTER DATABASE LocomotiveTracking SET COMPATIBILITY_LEVEL = 150"
```

## Key Features & Patterns

### Firmware Architecture
- **LILYGO T-SIM7600G-H Base**: Integrated ESP32 + 4G LTE + GPS foundation
- **Cellular communication**: 4G/3G/2G with automatic fallback - always available
- **Modular expansion**: Hot-swappable LoRa and satellite communication modules
- **Power management**: Railway power integration with 48-hour battery backup
- **Route-based profiles**: Adaptive reporting intervals and module activation based on location
- **Comprehensive OTA system**: Secure fleet-wide firmware updates via cellular

### OTA Update System
The firmware includes enterprise-grade OTA capabilities:
- **Fleet-wide updates**: Deploy to entire locomotive fleet via cellular
- **Security**: RSA signature verification with certificate-based authentication
- **Staged rollouts**: Canary deployments with automatic rollback protection
- **Real-time monitoring**: Update progress tracking and success reporting
- **Partition management**: Dual-partition setup with automatic failover

### Backend Services
- **Web API**: RESTful endpoints for device and fleet management
- **MQTT Integration**: Real-time telemetry ingestion from tracking devices
- **SignalR Hub**: Live dashboard updates and notifications
- **Background Services**: Data processing, maintenance, and alerting
- **Spatial Data**: MS SQL Server with geography extensions for route management

### Communication Strategy
1. **Cellular (Primary)**: 4G/3G/2G via private APN for 99%+ coverage
2. **LoRa (Optional)**: 868MHz for depot areas and cost optimization
3. **Satellite (Optional)**: Swarm/Iridium for remote routes (100% global coverage)
4. **Store & Forward**: Local buffering when all networks unavailable

## Configuration

### Firmware Configuration
Key configuration files in `firmware/config/`:
- `apn_settings.h`: Private APN configuration for cellular connectivity
- `routes.h`: Geographic boundaries and communication preferences
- `device_config.h`: Device-specific settings and debug levels

### Backend Configuration
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=LocomotiveTracking;Trusted_Connection=true",
    "MqttBroker": "mqtt://localhost:1883"
  },
  "MqttSettings": {
    "ClientId": "railway-backend",
    "Username": "railway",
    "QosLevel": 1
  }
}
```

## Testing Strategy

### Firmware Testing
- **Unit Tests**: Individual function and class validation
- **Integration Tests**: Hardware component interaction testing
- **Hardware-in-Loop**: Testing under actual railway conditions
- **OTA Testing**: Comprehensive update process validation

### Backend Testing
- **Unit Tests**: xUnit for business logic
- **Integration Tests**: Real database testing
- **API Tests**: WebApplicationFactory endpoint testing

### System Testing
- **End-to-end**: Complete data flow from device to dashboard
- **Failover Tests**: Network switching and recovery validation
- **Performance Tests**: Load testing for fleet-scale operations

## Safety & Standards

This is a **safety-critical railway system** that must comply with:
- Railway EMC standards (EN 50121)
- Vibration resistance (IEC 61373) 
- IP67 protection rating
- Temperature operation range: -20°C to +70°C
- 110V railway power integration

### Security Requirements
- TLS 1.3 for all communications
- RSA signature verification for OTA updates
- JWT authentication for API access
- Comprehensive audit logging
- No credential storage in code or commits

## Branching Strategy
- **main**: Production-ready code
- **develop**: Integration branch for features
- **feature/***: Individual feature development
- **hotfix/***: Critical production fixes
- **release/***: Release preparation

## Documentation
- **API Documentation**: Available at `/swagger` when running backend
- **Hardware Documentation**: See `hardware/README.md` for PCB designs
- **Implementation Guide**: Complete technical documentation in `docs/`
- **Research & Concepts**: Comprehensive analysis in `docs/concepts/` - covers 5 communication concepts evaluated and T-SIM selection rationale
- **Contributing Guidelines**: See `CONTRIBUTING.md` for development standards

## Important Notes
- Each tracking device requires unique Device ID configuration during deployment
- Route profiles automatically adjust communication method and reporting frequency
- System supports 100+ simultaneous devices with real-time updates
- OTA updates can be deployed to entire fleet or specific device groups
- All communications use MQTT over TLS for security and reliability