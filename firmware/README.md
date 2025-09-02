# Firmware

ESP32-based firmware for the LILYGO T-SIM7600G-H locomotive tracking system.

## Architecture

The firmware is designed with a modular architecture supporting:

- **Core tracking functionality** - GPS positioning and data logging
- **Cellular communication** - 4G/3G/2G connectivity with automatic fallback
- **Expansion modules** - Optional LoRa and satellite communication
- **Power management** - Railway power integration with battery backup
- **Configuration management** - Route-based profiles and settings

## Directory Structure

```
firmware/
├── core/              # Core application logic
│   ├── main.cpp       # Main application entry point
│   ├── gps_manager.cpp    # GPS positioning and tracking
│   ├── cellular_manager.cpp  # SIM7600G cellular modem control
│   ├── power_manager.cpp     # Power optimization and management
│   └── storage_manager.cpp   # SD card data logging
├── modules/           # Communication module drivers
│   ├── lora_module.cpp      # LoRa communication (optional)
│   ├── satellite_module.cpp # Satellite interface (optional)
│   └── module_interface.h   # Common module interface
├── config/            # Configuration management
│   ├── apn_settings.h       # Private APN configuration
│   ├── routes.h             # Route profiles and boundaries
│   └── device_config.h      # Device-specific settings
└── tests/             # Unit tests and hardware tests
    ├── test_gps.cpp         # GPS functionality tests
    ├── test_cellular.cpp    # Cellular communication tests
    └── test_integration.cpp # Full system integration tests
```

## Hardware Platform

- **Board**: LILYGO T-SIM7600G-H
- **MCU**: ESP32-WROVER (240MHz dual-core)
- **Cellular**: SIM7600G (4G LTE Cat-4, 3G, 2G)
- **GPS**: Integrated GNSS (GPS, GLONASS, BeiDou, Galileo)
- **Storage**: 32GB MicroSD card
- **Power**: 110V railway power with battery backup

## Key Features

### Adaptive Reporting
- **Stationary**: 10-minute intervals
- **Moving <20km/h**: 5-minute intervals  
- **Moving 20-60km/h**: 2-minute intervals
- **Moving >60km/h**: 1-minute intervals

### Communication Priority
1. **Cellular** (primary) - Real-time MQTT transmission
2. **LoRa** (optional) - Cost-effective depot communication
3. **Satellite** (optional) - Remote area coverage
4. **Store & Forward** - Local buffering when offline

### Route-Based Profiles
- **Urban Routes**: Cellular priority, 1-minute updates
- **Main Lines**: Cellular with LoRa backup, 2-minute updates
- **Remote Routes**: Satellite primary, 5-minute updates
- **Depot Operations**: LoRa only, 10-minute updates

## Development Environment

### Prerequisites
```bash
# Install PlatformIO
pip install platformio

# Install ESP32 toolchain
pio platform install espressif32
```

### Build Instructions
```bash
cd firmware/core
pio init --board esp32dev
pio lib install "SIM7600"
pio lib install "TinyGPSPlus"
pio lib install "ArduinoJson"
pio lib install "PubSubClient"
pio run
```

### Flash Firmware
```bash
pio run --target upload --upload-port /dev/ttyUSB0
```

## Configuration

### APN Settings
Edit `config/apn_settings.h`:
```cpp
const char* APN = "transnet.m2m";
const char* APN_USER = "railway";
const char* APN_PASS = "your_password";
```

### Route Profiles
Edit `config/routes.h` to define geographical boundaries and communication preferences for different route types.

### Device ID
Each device must be assigned a unique ID during deployment for tracking and identification.

## Testing

### Hardware Tests
- **GPS Signal Test**: Validate satellite reception and positioning accuracy
- **Cellular Test**: Verify network connectivity and data transmission
- **Power Test**: Confirm railway power integration and battery backup
- **Environmental Test**: Validate operation in temperature/vibration conditions

### Integration Tests
- **End-to-end Communication**: Test data flow from device to backend
- **Failover Tests**: Validate automatic network switching
- **Power Cycle Tests**: Ensure proper recovery after power interruption
- **Route Transition Tests**: Validate profile switching based on GPS location

## Deployment

1. **Configure device settings** for specific locomotive assignment
2. **Flash firmware** with appropriate configuration
3. **Install hardware** in locomotive electrical cabinet
4. **Validate connectivity** to backend systems
5. **Test tracking functionality** during trial runs

## Troubleshooting

### Common Issues
- **No GPS Fix**: Check antenna connection and sky view
- **Cellular Connection Failed**: Verify APN settings and SIM activation
- **Data Not Transmitting**: Check MQTT broker connectivity
- **High Power Consumption**: Review sleep mode configuration

### Debug Output
Enable serial debugging by setting `DEBUG_LEVEL` in `config/device_config.h`.

## Over-the-Air Updates

The firmware supports OTA updates via cellular connection for remote maintenance and feature updates.

---

*For detailed technical specifications, see the [Implementation Guide](../docs/implementation-guide.md).*