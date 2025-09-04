# Firmware

ESP32-based firmware for the LILYGO T-SIM7600G-H locomotive tracking system.

## Architecture

The firmware is designed with a modular architecture supporting:

- **Core tracking functionality** - GPS positioning and data logging
- **Enhanced User Interface** - 2.8" color TFT display, 4-button navigation, train management workflow
- **Train Management Integration** - Train number assignment, confirmation, and status display
- **Cellular communication** - 4G/3G/2G connectivity with automatic fallback
- **Expansion modules** - Optional LoRa and satellite communication
- **Power management** - Railway power integration with battery backup (updated for enhanced UI)
- **Configuration management** - Route-based profiles and settings

## Directory Structure

```
firmware/
├── core/              # Core application logic
│   ├── main.cpp       # Main application entry point
│   ├── gps_manager.cpp    # GPS positioning and tracking
│   ├── cellular_manager.cpp  # SIM7600G cellular modem control
│   ├── ui_manager.cpp     # Enhanced UI management (TFT display, buttons)
│   ├── gpio_expander.cpp  # MCP23017 GPIO expander driver
│   ├── train_manager.cpp  # Train assignment workflow and management
│   ├── power_manager.cpp     # Power optimization and management (updated for UI)
│   ├── storage_manager.cpp   # SD card data logging
│   ├── debug_manager.cpp     # Debug mode coordination and control
│   ├── wifi_ap_manager.cpp   # WiFi Access Point setup and management
│   └── web_server.cpp        # HTTP server and API endpoints
├── modules/           # Communication module drivers
│   ├── lora_module.cpp      # LoRa communication (optional)
│   ├── satellite_module.cpp # Satellite interface (optional)
│   └── module_interface.h   # Common module interface
├── drivers/           # Hardware drivers
│   ├── tft_display.cpp      # 2.8" TFT display driver
│   ├── button_matrix.cpp    # 4-button navigation driver
│   ├── led_controller.cpp   # Multi-color LED control
│   └── debug_led.cpp        # Debug-specific LED control
├── config/            # Configuration management
│   ├── apn_settings.h       # Private APN configuration
│   ├── routes.h             # Route profiles and boundaries
│   ├── device_config.h      # Device-specific settings
│   ├── ui_config.h          # UI settings, display configuration, button mappings
│   ├── gpio_config.h        # GPIO pin assignments and expander configuration
│   └── debug_config.h       # WiFi Access Point and debug mode configuration
├── docs/              # Firmware documentation
│   ├── debug-mode-guide.md     # Complete debug mode documentation
│   └── debug-api-reference.md  # Python/API interface documentation
└── tests/             # Unit tests and hardware tests
    ├── test_gps.cpp         # GPS functionality tests
    ├── test_cellular.cpp    # Cellular communication tests
    ├── test_ui.cpp          # Enhanced UI functionality tests
    ├── test_gpio_expander.cpp # GPIO expander tests
    ├── test_debug_mode.cpp  # Debug mode functionality tests
    └── test_integration.cpp # Full system integration tests
```

## Hardware Platform

- **Board**: LILYGO T-SIM7600G-H
- **MCU**: ESP32-WROVER (240MHz dual-core)
- **Cellular**: SIM7600G (4G LTE Cat-4, 3G, 2G)
- **GPS**: Integrated GNSS (GPS, GLONASS, BeiDou, Galileo)
- **Enhanced UI**: 2.8" color TFT display (320x240), 4-button navigation pad, multi-color status LEDs
- **GPIO Expander**: MCP23017 I2C 16-port expander for UI components
- **Storage**: 32GB MicroSD card
- **Power**: 110V railway power with battery backup (0.65W idle, 7.7W peak including UI + expander)

## Key Features

### Adaptive Reporting
- **Stationary**: 10-minute intervals
- **Moving <20km/h**: 5-minute intervals  
- **Moving 20-60km/h**: 2-minute intervals
- **Moving >60km/h**: 1-minute intervals

### Enhanced User Interface
- **2.8" TFT Display**: High-resolution color display (320x240)
- **4-Button Navigation**: Up, Down, Left, Right navigation pad + OK/Cancel buttons
- **Status LEDs**: Multi-color LEDs for GPS, cellular, train assignment, and system status
- **Menu System**: Intuitive menu navigation for device information and train status
- **Train Assignment Workflow**: Clear prompts for train number assignment and confirmation

### Communication Priority
1. **Cellular** (primary) - Real-time MQTT transmission with train assignment commands
2. **LoRa** (optional) - Cost-effective depot communication
3. **Satellite** (optional) - Remote area coverage
4. **Store & Forward** - Local buffering when offline

### Train Management Features
- **MQTT Command Processing**: Listen for train assignment commands from backend
- **User Confirmation Workflow**: Display train assignment request and prompt for confirmation
- **Train Status Display**: Show current assigned train number on main screen
- **Assignment History**: Local logging of train assignments for audit purposes
- **Auto-clear on Train Complete**: Automatic train number clearing on completion signal

### Route-Based Profiles
- **Urban Routes**: Cellular priority, 1-minute updates, train assignment enabled
- **Main Lines**: Cellular with LoRa backup, 2-minute updates, train assignment enabled
- **Remote Routes**: Satellite primary, 5-minute updates, train assignment enabled
- **Depot Operations**: LoRa only, 10-minute updates, train assignment disabled

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
pio lib install "TFT_eSPI"
pio lib install "XPT2046_Touchscreen"
pio lib install "Adafruit MCP23017 Arduino Library"
pio lib install "Wire"
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

### GPIO Configuration
Edit `config/gpio_config.h`:
```cpp
// ESP32 Direct GPIO Pin Assignments
#define TFT_CS_PIN       GPIO32   // TFT display chip select
#define TFT_DC_PIN       GPIO33   // TFT display data/command  
#define TFT_RST_PIN      GPIO25   // TFT display reset
#define TFT_BACKLIGHT    GPIO26   // TFT backlight control
// TFT uses shared SPI: GPIO14 (CLK), GPIO13 (MOSI), GPIO12 (MISO)

#define I2C_SDA_PIN      GPIO27   // I2C data line
#define I2C_SCL_PIN      GPIO14   // I2C clock line (shared with SPI CLK)

// GPIO Expander Configuration
#define I2C_GPIO_ADDR    0x20     // MCP23017 I2C address

// GPIO Expander Pin Assignments
#define EXP_BTN_UP       0        // Navigation button up
#define EXP_BTN_DOWN     1        // Navigation button down  
#define EXP_BTN_OK       2        // Confirmation button
#define EXP_BTN_CANCEL   3        // Cancel/back button
#define EXP_LED_GPS      4        // GPS status LED
#define EXP_LED_CELLULAR 5        // Cellular connectivity LED
#define EXP_LED_TRAIN    6        // Train assignment status LED
#define EXP_TRAIN_BUZZER 8        // Train assignment notification buzzer
```

### UI Configuration
Edit `config/ui_config.h`:
```cpp
// Display settings
#define TFT_WIDTH 320
#define TFT_HEIGHT 240
#define BACKGROUND_COLOR TFT_BLACK
#define TEXT_COLOR TFT_WHITE
#define HIGHLIGHT_COLOR TFT_CYAN

// Button debounce settings
#define BUTTON_DEBOUNCE_MS 50
#define BUTTON_LONG_PRESS_MS 1000

// Display timeout settings
#define DISPLAY_TIMEOUT_MS 300000  // 5 minutes auto-off
#define BACKLIGHT_DIM_LEVEL 50     // 0-255
```

### Device ID
Each device must be assigned a unique ID during deployment for tracking and identification.

## Testing

### Hardware Tests
- **GPS Signal Test**: Validate satellite reception and positioning accuracy
- **Cellular Test**: Verify network connectivity and data transmission
- **UI Hardware Test**: Test display, buttons, and LED functionality
- **Power Test**: Confirm railway power integration and battery backup (with UI load)
- **Environmental Test**: Validate operation in temperature/vibration conditions

### Integration Tests
- **End-to-end Communication**: Test data flow from device to backend including train assignments
- **Train Assignment Workflow**: Test complete train assignment and confirmation process
- **UI Interaction Tests**: Test all menu navigation and button responses
- **Failover Tests**: Validate automatic network switching
- **Power Cycle Tests**: Ensure proper recovery after power interruption (with UI state retention)
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
- **Display Not Working**: Check TFT display connections, SPI configuration, and power supply
- **Buttons Not Responsive**: Verify GPIO expander I2C connection and pin assignments
- **GPIO Expander Not Detected**: Check I2C wiring (SDA/SCL), pull-up resistors, and address conflicts
- **Train Assignments Not Working**: Check MQTT subscription topics and backend connectivity
- **Data Not Transmitting**: Check MQTT broker connectivity
- **High Power Consumption**: Review sleep mode configuration and display timeout settings
- **LED Status Not Working**: Verify GPIO expander initialization and pin configuration

### Debug Output
Enable serial debugging by setting `DEBUG_LEVEL` in `config/device_config.h`.

## Over-the-Air (OTA) Updates

The firmware provides comprehensive OTA update capabilities enabling secure, automated firmware deployment across the entire locomotive fleet via cellular connectivity.

### OTA Implementation Architecture

#### Core OTA Components
- **OTA Manager**: Handles update checking, downloading, and installation
- **Security Layer**: Firmware verification and cryptographic validation
- **Fleet Manager**: MQTT-based fleet-wide update coordination
- **Rollback System**: Automatic recovery from failed updates
- **Progress Reporting**: Real-time update status communication

#### Update Process Flow
1. **Discovery Phase**: Periodic version checking via HTTPS
2. **Validation Phase**: Server authentication and update verification
3. **Download Phase**: Secure firmware download to inactive partition
4. **Installation Phase**: Cryptographic verification and partition switch
5. **Verification Phase**: Self-test validation after reboot
6. **Commit Phase**: Update confirmation or automatic rollback

### PlatformIO Configuration for OTA

```ini
; platformio.ini - OTA-enabled configuration
[env:t-sim7600g-h-ota]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200

; OTA partition scheme
board_build.partitions = partitions_ota.csv

; OTA libraries
lib_deps = 
    TinyGSM@^0.11.7
    ArduinoJson@^6.21.3
    PubSubClient@^2.8.0
    ESP32httpUpdate@^2.1.145
    ArduinoHTTPSClient@^2.1.2

; OTA upload configuration
upload_protocol = espota
upload_port = 10.50.100.30
upload_flags = 
    --port=3232
    --auth=railway_ota_2024
```

### Partition Table for OTA

```csv
# partitions_ota.csv
# Name,   Type, SubType,  Offset,   Size,     Flags
nvs,      data, nvs,      0x9000,   0x5000,
otadata,  data, ota,      0xe000,   0x2000,
app0,     app,  ota_0,    0x10000,  0x180000,
app1,     app,  ota_1,    0x190000, 0x180000,
spiffs,   data, spiffs,   0x310000, 0x100000,
```

### Core OTA Implementation

#### Basic OTA Manager Class
```cpp
// ota_manager.h
#include <HTTPUpdate.h>
#include <TinyGsmClient.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

class RailwayOTAManager {
private:
    const char* update_server = "https://10.50.100.30";
    const char* current_version = FIRMWARE_VERSION;
    const char* device_id = DEVICE_ID;
    
    TinyGsmClient* gsm_client;
    PubSubClient* mqtt_client;
    
    bool update_available = false;
    String update_url;
    String update_version;
    
public:
    void begin(TinyGsmClient* client, PubSubClient* mqtt);
    bool checkForUpdates();
    void performUpdate();
    void handleFleetUpdate(char* topic, byte* payload, unsigned int length);
    bool validateFirmware(uint8_t* firmware, size_t size);
    void reportUpdateStatus(const char* status);
};
```

### Security Implementation

#### Secure Update Validation
```cpp
// security.cpp - Firmware verification
#include "mbedtls/pk.h"
#include "mbedtls/sha256.h"

class SecureOTA {
private:
    // RSA public key for firmware verification
    const char* public_key_pem = R"(
-----BEGIN PUBLIC KEY-----
[Your RSA Public Key for Firmware Signing]
-----END PUBLIC KEY-----
)";
    
public:
    bool verifyFirmwareSignature(const uint8_t* firmware, size_t size, 
                                const uint8_t* signature, size_t sig_len) {
        mbedtls_pk_context pk;
        mbedtls_pk_init(&pk);
        
        // Parse public key
        int ret = mbedtls_pk_parse_public_key(&pk, 
            (const unsigned char*)public_key_pem, 
            strlen(public_key_pem) + 1);
        if (ret != 0) return false;
        
        // Calculate firmware hash
        uint8_t hash[32];
        mbedtls_sha256(firmware, size, hash, 0);
        
        // Verify signature
        ret = mbedtls_pk_verify(&pk, MBEDTLS_MD_SHA256, 
                               hash, 32, signature, sig_len);
        
        mbedtls_pk_free(&pk);
        return (ret == 0);
    }
    
    bool validateFirmwareVersion(const char* new_version) {
        // Prevent downgrade attacks
        return (strcmp(new_version, current_version) > 0);
    }
};
```

### Fleet Management System

#### MQTT-Based Fleet Updates
```cpp
// fleet_manager.cpp
void FleetOTAManager::handleMqttMessage(char* topic, byte* payload, unsigned int length) {
    StaticJsonDocument<512> doc;
    deserializeJson(doc, payload, length);
    
    if (strstr(topic, "/update")) {
        const char* version = doc["version"];
        const char* url = doc["url"];
        const char* checksum = doc["md5"];
        int delay_seconds = doc["delay"] | 0;
        
        // Validate update command
        if (version && url && validateUpdateCommand(doc)) {
            scheduleUpdate(version, url, checksum, delay_seconds);
        }
    }
}

void FleetOTAManager::scheduleUpdate(const char* version, const char* url, 
                                   const char* checksum, int delay_seconds) {
    // Add random jitter to prevent thundering herd
    int jitter = random(0, 300); // 0-5 minutes
    unsigned long update_time = millis() + (delay_seconds + jitter) * 1000;
    
    // Store update parameters
    update_scheduled = true;
    update_scheduled_time = update_time;
    update_target_version = String(version);
    update_target_url = String(url);
    update_target_checksum = String(checksum);
    
    // Report scheduling
    reportUpdateStatus("scheduled");
}
```

### Rollback and Recovery

#### Automatic Rollback System
```cpp
// rollback.cpp
#include "esp_ota_ops.h"

class RollbackManager {
public:
    void markUpdateValid() {
        // Mark current partition as valid
        esp_ota_mark_app_valid_cancel_rollback();
        Serial.println("OTA: Update marked as valid");
    }
    
    void triggerRollback() {
        // Force rollback to previous partition
        esp_ota_mark_app_invalid_rollback_and_reboot();
    }
    
    bool validateNewFirmware() {
        // Perform comprehensive self-tests
        bool gps_ok = testGPS();
        bool cellular_ok = testCellular();
        bool mqtt_ok = testMQTT();
        bool sensors_ok = testSensors();
        
        return gps_ok && cellular_ok && mqtt_ok && sensors_ok;
    }
    
private:
    bool testGPS() {
        // Test GPS functionality
        return true; // Implementation specific
    }
    
    bool testCellular() {
        // Test cellular connectivity
        return modem.isNetworkConnected();
    }
    
    bool testMQTT() {
        // Test MQTT connection
        return mqtt_client->connected();
    }
    
    bool testSensors() {
        // Test hardware sensors
        return true; // Implementation specific
    }
};
```

### Update Deployment Strategies

#### Staged Rollout Implementation
```cpp
// deployment.cpp
enum class DeploymentStrategy {
    IMMEDIATE,    // All devices immediately
    CANARY,      // 5% first, then remainder
    STAGED,      // Deploy in time-based groups
    GEOGRAPHIC   // Deploy by geographic regions
};

class DeploymentManager {
public:
    void deployUpdate(DeploymentStrategy strategy, const char* version) {
        switch (strategy) {
            case DeploymentStrategy::CANARY:
                deployCanary(version);
                break;
            case DeploymentStrategy::STAGED:
                deployStaged(version);
                break;
            case DeploymentStrategy::GEOGRAPHIC:
                deployGeographic(version);
                break;
            default:
                deployImmediate(version);
        }
    }
    
private:
    void deployCanary(const char* version) {
        // Deploy to 5% test group first
        publishUpdate("fleet/canary/update", version);
        
        // Wait 24 hours, then deploy to remainder if successful
        scheduleFollowUp("fleet/main/update", version, 86400000);
    }
};
```

### Monitoring and Diagnostics

#### Update Progress Reporting
```cpp
// monitoring.cpp
class UpdateMonitor {
private:
    unsigned long update_start_time;
    size_t total_size;
    size_t downloaded_bytes;
    
public:
    void onUpdateStart() {
        update_start_time = millis();
        reportProgress("started", 0);
    }
    
    void onUpdateProgress(size_t current, size_t total) {
        downloaded_bytes = current;
        total_size = total;
        
        int percent = (current * 100) / total;
        if (percent % 10 == 0) { // Report every 10%
            reportProgress("downloading", percent);
        }
    }
    
    void onUpdateComplete(bool success) {
        unsigned long duration = millis() - update_start_time;
        
        StaticJsonDocument<256> report;
        report["device_id"] = device_id;
        report["success"] = success;
        report["duration_ms"] = duration;
        report["size_bytes"] = total_size;
        
        String json;
        serializeJson(report, json);
        mqtt_client->publish("fleet/update/report", json.c_str());
    }
    
private:
    void reportProgress(const char* phase, int percent) {
        StaticJsonDocument<128> status;
        status["device_id"] = device_id;
        status["phase"] = phase;
        status["percent"] = percent;
        
        String json;
        serializeJson(status, json);
        mqtt_client->publish("fleet/update/status", json.c_str());
    }
};
```

### Testing Framework

#### OTA Testing Suite
```cpp
// test_ota.cpp
void testOTAFunctionality() {
    Serial.println("=== OTA Test Suite ===");
    
    // Test 1: Partition validation
    const esp_partition_t* running = esp_ota_get_running_partition();
    const esp_partition_t* update = esp_ota_get_next_update_partition(NULL);
    assert(running != update);
    Serial.println("✓ OTA partitions configured correctly");
    
    // Test 2: Network connectivity
    assert(modem.isGprsConnected());
    Serial.println("✓ Cellular connection active");
    
    // Test 3: Server reachability
    HTTPClient http;
    http.begin(*gsm_client, "https://10.50.100.30/health");
    assert(http.GET() == 200);
    Serial.println("✓ Update server reachable");
    
    // Test 4: MQTT connectivity
    assert(mqtt_client.connected());
    Serial.println("✓ MQTT fleet management active");
    
    // Test 5: Security validation
    SecureOTA security;
    assert(security.validateFirmwareVersion("999.999.999"));
    Serial.println("✓ Security validation functional");
    
    Serial.println("=== All OTA tests passed ===");
}
```

### Production Configuration

#### Build Flags for Production OTA
```cpp
// config/ota_config.h
#ifdef PRODUCTION_BUILD
    #define OTA_SERVER_URL "https://10.50.100.30"
    #define MQTT_BROKER "10.50.100.10"
    #define OTA_CHECK_INTERVAL 3600000  // 1 hour
    #define ENABLE_SECURE_OTA true
    #define ENABLE_SIGNATURE_VERIFICATION true
#else
    #define OTA_SERVER_URL "http://192.168.1.100"
    #define MQTT_BROKER "192.168.1.100"
    #define OTA_CHECK_INTERVAL 300000   // 5 minutes
    #define ENABLE_SECURE_OTA false
    #define ENABLE_SIGNATURE_VERIFICATION false
#endif

// Fleet management topics
#define TOPIC_FLEET_ALL_UPDATE "fleet/all/update"
#define TOPIC_FLEET_DEVICE_UPDATE "fleet/" DEVICE_ID "/update"
#define TOPIC_FLEET_GROUP_UPDATE "fleet/group1/update"
#define TOPIC_FLEET_STATUS "fleet/status/" DEVICE_ID
#define TOPIC_FLEET_UPDATE_REPORT "fleet/update/report"
```

## Debug Mode - WiFi Access Point Data Extraction

The firmware includes a comprehensive debug mode that creates a WiFi Access Point for field data extraction and debugging. This enables technicians to access on-board data without physical connections, supporting both manual browser access and automated Python scripts.

### Debug Mode Overview

**Purpose**: Field-accessible data extraction and system diagnostics
**Activation**: Hardware trigger (OK+Cancel buttons for 3 seconds) or UI menu
**Interface**: WiFi Access Point with web server at 192.168.4.1
**Security**: Daily-rotating passwords and session timeouts

### WiFi Access Point Configuration

```cpp
// Debug WiFi AP Settings
SSID: "LOCO-{DEVICE_ID}-DEBUG"    // e.g., "LOCO-001234-DEBUG"
Password: "{DEVICE_ID}{YYYYMMDD}" // e.g., "00123420250904" (daily rotation)
IP Address: 192.168.4.1
Gateway: 192.168.4.1
Subnet: 255.255.255.0
Channel: 6 (2.4GHz)
Max Clients: 4 concurrent connections
Session Timeout: 30 minutes inactivity
Auto-Exit: 2 hours maximum duration
```

### Activation Methods

#### Hardware Activation (Primary)
1. **Button Combination**: Hold OK+Cancel buttons simultaneously for 3 seconds
2. **Display Confirmation**: Enhanced UI shows debug mode activation screen
3. **LED Indication**: System LED changes to blue flashing pattern
4. **Audio Feedback**: Three beeps confirm debug mode activation

#### UI Menu Activation (Secondary)
1. Navigate to Main Menu → Settings → Debug Mode
2. Select "Activate Debug Mode"
3. Confirm activation on confirmation screen
4. Debug mode starts with visual and audio feedback

### Web Interface Features

#### Browser Access (http://192.168.4.1)
- **Landing Page**: Device information, storage statistics, connection status
- **File Browser**: Hierarchical SD card explorer with download capabilities
- **Log Viewer**: Real-time log streaming and historical log access
- **System Status**: Live GPS, cellular, train assignment, and sensor data
- **Bulk Download**: ZIP archive creation for selected files/folders
- **Configuration Viewer**: Current device settings (read-only access)

#### REST API Endpoints
```bash
# System Information
GET /api/system/info          # Device info, uptime, version
GET /api/system/status        # Real-time system status
GET /api/system/storage       # SD card usage statistics

# File Operations
GET /api/files                # List all files and directories
GET /api/files/{path}         # Download specific file
POST /api/files/archive       # Create ZIP archive of selected files
GET /api/files/tree           # Complete directory tree structure

# Log Access
GET /api/logs                 # Available log files
GET /api/logs/{type}          # Specific log type (system, gps, cellular, train)
GET /api/logs/stream          # WebSocket for real-time log streaming

# Configuration
GET /api/config               # Current device configuration (read-only)
GET /api/config/routes        # Route profiles and boundaries
GET /api/config/gpio          # GPIO pin assignments and expander status
```

### SD Card File Organization

Debug mode provides access to organized data structure:

```
/data/
├── logs/
│   ├── system/              # System logs by date (YYYY-MM-DD.log)
│   ├── gps/                 # GPS tracking data and positioning logs
│   ├── cellular/            # Cellular connection and network logs
│   ├── train/               # Train assignment history and workflow logs
│   └── debug/               # Debug session logs and access history
├── config/
│   ├── device.json          # Device configuration backup
│   ├── routes.json          # Route profile data and boundaries
│   ├── gpio_state.json      # GPIO and expander configuration
│   └── calibration.json     # Sensor calibration and offset data
├── exports/
│   ├── daily/               # Daily data exports and summaries
│   └── sessions/            # Debug session data exports
└── temp/
    └── debug_archives/      # Temporary ZIP files for bulk downloads
```

### Python API Integration

```python
import requests
import json
from datetime import datetime

# Connect to debug device
base_url = "http://192.168.4.1/api"

# Get device information
response = requests.get(f"{base_url}/system/info")
device_info = response.json()
print(f"Device: {device_info['device_id']}")
print(f"Uptime: {device_info['uptime_hours']} hours")

# Download specific log file
log_response = requests.get(f"{base_url}/logs/system")
with open(f"system_log_{datetime.now().strftime('%Y%m%d')}.log", 'wb') as f:
    f.write(log_response.content)

# Create and download archive of all GPS data
archive_request = {
    "paths": ["/data/logs/gps/"],
    "archive_name": "gps_data_export"
}
archive_response = requests.post(f"{base_url}/files/archive", json=archive_request)
archive_url = archive_response.json()['download_url']

# Download the created archive
archive_data = requests.get(f"{base_url}{archive_url}")
with open("gps_data_export.zip", 'wb') as f:
    f.write(archive_data.content)
```

### Enhanced UI Integration

Debug mode integrates seamlessly with the existing Enhanced UI system:

#### Debug Mode Status Screen
```
┌─────────────────────────────────────┐
│ 📶 DEBUG MODE ACTIVE                │
│                                     │
│ WiFi AP: LOCO-001234-DEBUG          │
│ Password: 00123420250904            │
│ IP: 192.168.4.1                    │
│                                     │
│ Connected Clients: 1/4              │
│ Data Downloaded: 127.3 MB          │
│ Session Time: 15:23 / 120:00       │
│                                     │
│ Web: http://192.168.4.1             │
│                [EXIT]    [STATUS]   │
└─────────────────────────────────────┘
```

#### Status LED Integration
- **System LED**: Blue flashing pattern during debug mode (vs. normal green)
- **Debug LED**: New LED via GPIO expander (pin 9)
  - Green solid: WiFi AP active, no clients connected
  - Orange solid: Client connected, data access in progress
  - Red flashing: Debug mode error or timeout warning

### Security Features

#### Access Control
- **Dynamic Passwords**: Daily rotation based on device ID + date
- **Session Management**: 30-minute inactivity timeout, 2-hour maximum duration
- **Client Limits**: Maximum 4 concurrent connections
- **IP Restrictions**: Optional MAC address whitelist for authorized devices

#### Data Protection
- **Read-Only Access**: No configuration modification via debug interface
- **Selective Exposure**: Excludes sensitive files (keys, credentials, certificates)
- **Audit Logging**: All debug sessions and file access logged to `/data/logs/debug/`
- **Secure Deletion**: Option to securely wipe debug logs after export
- **Access Verification**: Debug session summary shown on Enhanced UI after exit


### Troubleshooting Debug Mode

#### Common Issues and Solutions

**WiFi AP Not Visible**:
- Verify debug mode activation (check system LED pattern)
- Ensure WiFi is enabled on connecting device
- Check for channel conflicts (AP uses channel 6)
- Verify antenna connections and placement

**Cannot Connect to WiFi AP**:
- Confirm correct password format: {DEVICE_ID}{YYYYMMDD}
- Check device limit (maximum 4 concurrent clients)
- Verify client device WiFi compatibility (2.4GHz only)
- Try reactivating debug mode to reset AP

**Web Interface Not Loading**:
- Confirm connection to debug WiFi network
- Navigate to http://192.168.4.1 (not https)
- Check client device firewall settings
- Try different browser or clear cache

**File Downloads Failing**:
- Verify sufficient storage space on client device
- Check file permissions and SD card health
- Monitor session timeout (30-minute limit)
- Try smaller file selections for bulk downloads

**Debug Mode Won't Activate**:
- Ensure buttons held for full 3 seconds
- Check GPIO expander I2C communication
- Verify Enhanced UI functionality
- Review system error logs for conflicts

#### Debug Mode Testing

```bash
# Test debug mode activation
pio test -e debug-mode-activation

# Test WiFi AP functionality  
pio test -e debug-wifi-ap

# Test web server endpoints
pio test -e debug-web-server

# Test file access and downloads
pio test -e debug-file-operations

# Test security and session management
pio test -e debug-security

# Full debug mode integration test
pio test -e debug-mode-full
```

### Development Integration

Debug mode requires additional PlatformIO libraries:

```bash
# Install debug mode dependencies
pio lib install "WiFi"           # ESP32 WiFi library
pio lib install "WebServer"      # Basic HTTP server
pio lib install "ESPAsyncWebServer" # Async web server for better performance  
pio lib install "AsyncTCP"       # TCP library for async server
pio lib install "ArduinoJson"    # JSON processing (already included)
```

### Field Usage Workflow

#### Technician Procedure
1. **Approach Device**: Locate locomotive tracking unit
2. **Activate Debug Mode**: Hold OK+Cancel buttons for 3 seconds
3. **Confirm Activation**: Check display for WiFi credentials
4. **Connect Laptop**: Join "LOCO-{ID}-DEBUG" network with displayed password
5. **Access Interface**: Navigate to http://192.168.4.1 in browser
6. **Extract Data**: Download required logs, configuration, or use Python scripts
7. **Exit Cleanly**: Press EXIT button or wait for auto-timeout
8. **Verify Completion**: Confirm normal operation resumed

#### Python Script Automation
```python
#!/usr/bin/env python3
"""
Automated locomotive data extraction script
Usage: python3 extract_data.py <device_ip> [--logs] [--config] [--all]
"""

import argparse
import requests
import os
from datetime import datetime

def extract_locomotive_data(device_ip, options):
    base_url = f"http://{device_ip}/api"
    
    # Create timestamped output directory
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    output_dir = f"locomotive_data_{timestamp}"
    os.makedirs(output_dir, exist_ok=True)
    
    if options.logs or options.all:
        print("Downloading log files...")
        download_logs(base_url, output_dir)
    
    if options.config or options.all:
        print("Downloading configuration...")
        download_config(base_url, output_dir)
    
    print(f"Data extraction complete. Files saved to: {output_dir}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Extract locomotive data via debug mode")
    parser.add_argument("device_ip", default="192.168.4.1", help="Device IP address")
    parser.add_argument("--logs", action="store_true", help="Download log files")
    parser.add_argument("--config", action="store_true", help="Download configuration")
    parser.add_argument("--all", action="store_true", help="Download everything")
    
    args = parser.parse_args()
    extract_locomotive_data(args.device_ip, args)
```

---

*For complete debug mode specifications, see [debug-mode-guide.md](debug-mode-guide.md) and [debug-api-reference.md](debug-api-reference.md).*

---

*For detailed technical specifications, see the [Implementation Guide](../docs/implementation-guide.md).*