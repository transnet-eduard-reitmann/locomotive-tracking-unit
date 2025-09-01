# South African Railway Locomotive GPS Tracking System
## Implementation Guide - Cellular-Base with Modular Extensions

---

## Executive Summary

This document provides comprehensive technical documentation for implementing a GPS tracking system using the LILYGO T-SIM7600G-H as the base platform with optional expansion modules. The solution provides immediate cellular connectivity with the flexibility to add LoRa or satellite communication for areas with poor cellular coverage.

**Key Innovation:** Unlike a fixed hybrid system, this approach starts with proven cellular technology and adds modules only where operationally required, optimizing both deployment speed and total cost of ownership.

---

## 1. System Architecture

### 1.1 Core Design Principles
- **Cellular-first approach** - Immediate real-time tracking capability
- **Modular expansion** - Add communication methods as needed
- **Private network security** - APN configuration for company network
- **Graceful degradation** - Store-and-forward when offline
- **Single platform** - One hardware design, multiple configurations

### 1.2 Network Architecture

```
                FIELD DEVICES                           PRIVATE NETWORK
┌─────────────────────────────────────────┐    ┌──────────────────────────┐
│                                         │    │                          │
│  Locomotive Unit #1 (Urban)            │    │   Company Data Center    │
│  └─ T-SIM7600G-H (Cellular only)       │───►│                          │
│                                         │    │   ┌──────────────────┐  │
│  Locomotive Unit #2 (Depot)            │    │   │  MS SQL Server   │  │
│  └─ T-SIM7600G-H + LoRa Module         │───►│   │  Spatial DB      │  │
│                                         │    │   └──────────────────┘  │
│  Locomotive Unit #3 (Remote)           │    │                          │
│  └─ T-SIM7600G-H + Satellite Module    │───►│   ┌──────────────────┐  │
│                                         │    │   │  MQTT Broker     │  │
│  Locomotive Unit #4 (Cross-border)     │    │   │  (Mosquitto)     │  │
│  └─ T-SIM7600G-H + Sat + LoRa         │───►│   └──────────────────┘  │
│                                         │    │                          │
└─────────────────────────────────────────┘    │   ┌──────────────────┐  │
                    │                           │   │  Web Dashboard   │  │
                    │                           │   │  (.NET Core)     │  │
            [Private APN Tunnel]                │   └──────────────────┘  │
                    │                           │                          │
                    ▼                           │   ┌──────────────────┐  │
         ┌──────────────────┐                  │   │  Active Directory│  │
         │  Cellular Network │                  │   │  Authentication  │  │
         │  MTN/Vodacom APN  │─────────────────│   └──────────────────┘  │
         └──────────────────┘                  │                          │
                                               └──────────────────────────┘
```

### 1.3 Communication Priority Logic

```cpp
// Automatic selection based on availability and cost
Priority 1: Cellular (if signal > -100 dBm)
Priority 2: LoRa (if gateway in range and cellular weak)
Priority 3: Satellite (if no terrestrial options)
Priority 4: Store & Forward (buffer until connection available)
```

---

## 2. Hardware Design

### 2.1 Base Configuration

| Component | Model | Unit Price (ZAR) | Quantity | Total | Purpose |
|-----------|--------|------------------|----------|--------|----------|
| **Main Board** | LILYGO T-SIM7600G-H | R1,500 | 1 | R1,500 | ESP32 + 4G LTE + GPS |
| **External Antenna** | 4G/GPS Combo Antenna | R200 | 1 | R200 | Enhanced reception |
| **Power Supply** | Mean Well DDR-30G-5 | R350 | 1 | R350 | 9-36V to 5V DC-DC |
| **Backup Battery** | 18650 Li-ion x2 | R120 | 1 | R120 | 48-hour backup |
| **Battery Management** | TP4056 + Protection | R50 | 1 | R50 | Charging circuit |
| **Storage** | 32GB Industrial SD | R150 | 1 | R150 | Local data logging |
| **Enclosure** | IP67 Railway Box | R350 | 1 | R350 | Environmental protection |
| **Expansion Board** | Custom PCB | R200 | 1 | R200 | Module interface |
| **Connectors** | M12, terminal blocks | R100 | 1 | R100 | Robust connections |
| **Surge Protection** | TVS diodes, filters | R100 | 1 | R100 | Railway EMC |
| **TOTAL BASE** | | | | **R3,120** | **Complete base unit** |

### 2.2 Expansion Modules

#### 2.2.1 LoRa Module (Optional)
| Component | Model | Price | Purpose |
|-----------|--------|-------|---------|
| **LoRa Chip** | SX1276 868MHz | R180 | Long-range communication |
| **Antenna** | 3dBi 868MHz | R70 | Optimized for SA ISM band |
| **Interface** | SPI connection | R50 | Connection to main board |
| **Total** | | **R300** | LoRa capability |

#### 2.2.2 Satellite Module (Optional)
| Component | Model | Price | Purpose |
|-----------|--------|-------|---------|
| **Swarm M138** | Satellite modem | R3,200 | Global coverage |
| **VHF Antenna** | Included | R0 | Satellite communication |
| **Interface** | UART connection | R50 | Connection to main board |
| **Total** | | **R3,250** | Satellite capability |

### 2.3 Pin Assignments

```cpp
// T-SIM7600G-H Pin Assignments with Expansion Support

// Built-in peripherals (already connected internally)
// GPS: Connected via UART to SIM7600G
// Cellular: Integrated in SIM7600G
// SD Card: GPIO5 (CS), GPIO18 (CLK), GPIO19 (MISO), GPIO23 (MOSI)

// Available expansion pins on ESP32
#define LORA_CS_PIN      GPIO15   // LoRa chip select
#define LORA_RST_PIN     GPIO4    // LoRa reset
#define LORA_IRQ_PIN     GPIO2    // LoRa interrupt
#define LORA_SCK_PIN     GPIO14   // SPI clock (shared)
#define LORA_MISO_PIN    GPIO12   // SPI MISO (shared)
#define LORA_MOSI_PIN    GPIO13   // SPI MOSI (shared)

#define SAT_TX_PIN       GPIO16   // Satellite UART TX
#define SAT_RX_PIN       GPIO17   // Satellite UART RX
#define SAT_EN_PIN       GPIO21   // Satellite enable
#define SAT_STATUS_PIN   GPIO22   // Satellite status

#define I2C_SDA_PIN      GPIO26   // I2C for sensors
#define I2C_SCL_PIN      GPIO27   // I2C for sensors

#define STATUS_LED       GPIO25   // Status indicator
#define USER_BUTTON      GPIO0    // Configuration button

#define BATTERY_ADC      GPIO35   // Battery voltage monitor
#define EXTERNAL_INT     GPIO34   // External interrupt
```

### 2.4 Power Management

```
Railway Power (24-110V DC) ──┬──> DC-DC Converter (5V/3A)
                             │         │
                             │         ├──> T-SIM7600G-H (5V)
                             │         ├──> Expansion Modules
                             │         └──> Battery Charger
                             │                    │
                             └──> Monitoring <────┤
                                                  │
                                           18650 Battery Pack
                                           (7.4V, 6800mAh)
```

**Power Consumption:**
- **Idle:** 80mA @ 5V (0.4W)
- **GPS Active:** 150mA @ 5V (0.75W)
- **Cellular TX:** 1.5A @ 5V peak (7.5W)
- **LoRa TX:** 120mA @ 3.3V (0.4W)
- **Satellite TX:** 1A @ 5V (5W)

**Battery Life (6800mAh @ 7.4V = 50Wh):**
- **Cellular only:** 48 hours continuous operation
- **With LoRa:** 45 hours
- **With Satellite:** 36 hours

---

## 3. Communication Protocols

### 3.1 Private APN Configuration

```cpp
// APN Settings for Private Network Access
const char* APN = "transnet.m2m";        // Private APN name
const char* APN_USER = "railway";        // APN username
const char* APN_PASS = "encrypted_pwd";  // APN password

// Network Configuration
const char* MQTT_SERVER = "10.50.100.10";  // Internal MQTT broker
const int MQTT_PORT = 8883;                // TLS port
const char* SQL_SERVER = "10.50.100.20";   // MS SQL Server (via API)
```

### 3.2 Data Transmission Format

#### 3.2.1 MQTT Message Structure
```json
{
  "deviceId": "LOCO-001",
  "timestamp": "2025-09-01T10:30:45Z",
  "position": {
    "latitude": -26.195246,
    "longitude": 28.034088,
    "altitude": 1680.5,
    "accuracy": 2.5,
    "speed": 45.6,
    "heading": 127.3
  },
  "telemetry": {
    "satellites": 12,
    "hdop": 0.9,
    "signalStrength": -75,
    "network": "Vodacom-4G",
    "battery": 85,
    "temperature": 28.5
  },
  "modules": {
    "cellular": "active",
    "lora": "standby",
    "satellite": "not_installed"
  }
}
```

#### 3.2.2 LoRa Message Format (Compressed)
```cpp
struct LoRaPacket {
    uint16_t deviceId;        // 2 bytes
    uint32_t timestamp;       // 4 bytes (epoch)
    int32_t latitude;         // 4 bytes (scaled integer)
    int32_t longitude;        // 4 bytes (scaled integer)
    uint16_t speedHeading;    // 2 bytes (packed)
    uint8_t status;           // 1 byte (bit flags)
    uint8_t battery;          // 1 byte (percentage)
};  // Total: 18 bytes
```

### 3.3 Store-and-Forward Protocol

```cpp
class DataManager {
    void handleDataTransmission(TelemetryData& data) {
        // Always store locally first
        sdCard.append(data);
        
        // Try primary communication
        if (cellular.isConnected()) {
            if (cellular.send(data)) {
                sdCard.markAsSent(data);
                return;
            }
        }
        
        // Try secondary if available
        if (loraModule.isInstalled() && loraModule.gatewayInRange()) {
            if (loraModule.send(compressData(data))) {
                sdCard.markAsSent(data);
                return;
            }
        }
        
        // Queue for retry
        retryQueue.add(data);
    }
};
```

---

## 4. Software Architecture

### 4.1 Embedded Firmware Structure

```
firmware/
├── src/
│   ├── main.cpp                 // Main application loop
│   ├── config/
│   │   ├── apn_settings.h       // Private APN configuration
│   │   ├── routes.h             // Route profiles
│   │   └── modules.h            // Module configurations
│   ├── core/
│   │   ├── gps_manager.cpp      // GPS handling
│   │   ├── cellular_manager.cpp // SIM7600G control
│   │   ├── power_manager.cpp    // Power optimization
│   │   └── storage_manager.cpp  // SD card operations
│   ├── modules/
│   │   ├── lora_module.cpp      // LoRa communication
│   │   ├── satellite_module.cpp // Satellite interface
│   │   └── module_interface.h   // Common interface
│   ├── communication/
│   │   ├── mqtt_client.cpp      // MQTT protocol
│   │   ├── data_compression.cpp // Message optimization
│   │   └── encryption.cpp       // TLS/security
│   └── utils/
│       ├── diagnostics.cpp      // Self-test routines
│       ├── ota_update.cpp       // Firmware updates
│       └── watchdog.cpp         // System monitoring
├── lib/
│   ├── TinyGPSPlus/            // GPS parsing
│   ├── PubSubClient/           // MQTT library
│   ├── LoRa/                   // LoRa library
│   └── ArduinoJson/            // JSON handling
└── platformio.ini              // Build configuration
```

### 4.2 Core Application Logic

```cpp
class LocomotiveTracker {
private:
    // Core components
    GPSManager gps;
    CellularManager cellular;
    StorageManager storage;
    PowerManager power;
    
    // Optional modules
    LoRaModule* lora = nullptr;
    SatelliteModule* satellite = nullptr;
    
    // Configuration
    RouteProfile currentRoute;
    DeviceConfig config;
    
public:
    void setup() {
        // Initialize base system
        Serial.begin(115200);
        initializeHardware();
        
        // Configure private APN
        cellular.configure(APN, APN_USER, APN_PASS);
        cellular.connect();
        
        // Detect expansion modules
        detectModules();
        
        // Load configuration
        loadConfiguration();
        
        // Start tracking
        gps.begin();
        
        // Register with backend
        registerDevice();
    }
    
    void loop() {
        // Collect telemetry
        TelemetryData data = collectTelemetry();
        
        // Determine update interval based on movement
        uint32_t interval = calculateInterval(data.speed);
        
        // Store locally
        storage.log(data);
        
        // Transmit using best available method
        transmitData(data);
        
        // Power management
        if (interval > 60000) {
            power.enterDeepSleep(interval);
        } else {
            delay(interval);
        }
    }
    
private:
    void detectModules() {
        // Check for LoRa module on SPI bus
        if (checkSPIDevice(LORA_CS_PIN)) {
            lora = new LoRaModule(LORA_CS_PIN, LORA_RST_PIN, LORA_IRQ_PIN);
            lora->initialize();
            Serial.println("LoRa module detected");
        }
        
        // Check for Satellite module on UART
        if (checkUARTDevice(SAT_TX_PIN, SAT_RX_PIN)) {
            satellite = new SatelliteModule(SAT_TX_PIN, SAT_RX_PIN);
            satellite->initialize();
            Serial.println("Satellite module detected");
        }
    }
    
    uint32_t calculateInterval(float speed) {
        if (speed < 1.0) return 600000;   // 10 min when stationary
        if (speed < 20.0) return 300000;  // 5 min when slow
        if (speed < 60.0) return 120000;  // 2 min when moderate
        return 60000;                     // 1 min when fast
    }
    
    void transmitData(TelemetryData& data) {
        bool transmitted = false;
        
        // Try cellular first (private APN)
        if (cellular.isConnected()) {
            String json = serializeToJson(data);
            if (mqtt.publish("telemetry/locomotive", json)) {
                transmitted = true;
                updateDashboard(data);
            }
        }
        
        // Fallback to LoRa if available and cellular failed
        if (!transmitted && lora && lora->gatewayInRange()) {
            LoRaPacket packet = compressData(data);
            if (lora->send(packet)) {
                transmitted = true;
            }
        }
        
        // Last resort: satellite (if installed)
        if (!transmitted && satellite && data.priority == HIGH) {
            if (satellite->send(data)) {
                transmitted = true;
            }
        }
        
        // Queue for retry if all failed
        if (!transmitted) {
            storage.queueForRetry(data);
        }
    }
};
```

### 4.3 Backend Architecture

```csharp
// .NET Core Web API Structure
namespace RailwayTracking.API
{
    public class TelemetryController : ControllerBase
    {
        private readonly SqlContext _db;
        private readonly IHubContext<TrackingHub> _hub;
        
        [HttpPost("telemetry")]
        public async Task<IActionResult> ReceiveTelemetry(TelemetryData data)
        {
            // Validate device authentication (via APN)
            if (!IsValidDevice(data.DeviceId))
                return Unauthorized();
            
            // Store in SQL Server
            var entity = MapToEntity(data);
            _db.LocationHistory.Add(entity);
            await _db.SaveChangesAsync();
            
            // Update real-time dashboard
            await _hub.Clients.All.SendAsync("LocationUpdate", data);
            
            // Check geofencing rules
            await CheckGeofencing(data);
            
            return Ok();
        }
    }
}
```

---

## 5. Database Schema

### 5.1 MS SQL Server Tables

```sql
-- Core tracking tables for private network deployment
CREATE TABLE Devices (
    DeviceID INT IDENTITY PRIMARY KEY,
    IMEI NVARCHAR(20) UNIQUE,
    LocomotiveNumber NVARCHAR(50),
    InstallDate DATETIME2,
    ModulesInstalled NVARCHAR(100),
    IsActive BIT DEFAULT 1,
    LastSeen DATETIME2,
    CurrentRoute NVARCHAR(100)
);

CREATE TABLE LocationHistory (
    ID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    Timestamp DATETIME2,
    Position GEOGRAPHY,  -- Spatial data type
    Speed FLOAT,
    Heading FLOAT,
    Altitude FLOAT,
    Satellites INT,
    HDOP FLOAT,
    CommunicationMethod NVARCHAR(20), -- 'Cellular', 'LoRa', 'Satellite'
    SignalStrength INT,
    Battery INT,
    Temperature FLOAT,
    INDEX IX_Location_Time (DeviceID, Timestamp) INCLUDE (Position)
);

CREATE TABLE NetworkEvents (
    EventID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    Timestamp DATETIME2,
    EventType NVARCHAR(50),
    Details NVARCHAR(500),
    NetworkType NVARCHAR(20),
    SignalQuality INT
);

CREATE TABLE Routes (
    RouteID INT IDENTITY PRIMARY KEY,
    RouteName NVARCHAR(100),
    RouteGeometry GEOGRAPHY,  -- Spatial line
    PreferredComm NVARCHAR(20),
    UpdateInterval INT,
    GeofenceBuffer INT  -- meters
);

-- Stored procedure for efficient spatial queries
CREATE PROCEDURE GetLocomotivesInArea
    @Area GEOGRAPHY,
    @TimeWindow INT = 60  -- minutes
AS
BEGIN
    SELECT 
        d.LocomotiveNumber,
        l.Position.STAsText() as CurrentPosition,
        l.Speed,
        l.Heading,
        l.Timestamp,
        l.CommunicationMethod
    FROM LocationHistory l
    INNER JOIN Devices d ON l.DeviceID = d.DeviceID
    WHERE l.Position.STIntersects(@Area) = 1
        AND l.Timestamp > DATEADD(MINUTE, -@TimeWindow, GETUTCDATE())
        AND l.ID IN (
            SELECT MAX(ID) FROM LocationHistory 
            GROUP BY DeviceID
        )
END
```

---

## 6. Implementation Timeline

### 6.1 Phase 1: Cellular Deployment (Months 1-6)

**Month 1-2: Infrastructure Setup**
- Configure private APN with MTN/Vodacom
- Setup MS SQL Server database
- Deploy MQTT broker on company network
- Develop base firmware

**Month 3-4: Pilot Testing**
- Deploy 10 units on test locomotives
- Validate private network connectivity
- Test data flow to SQL Server
- Refine power management

**Month 5-6: Production Rollout**
- Manufacture 100 base units
- Installation on priority locomotives
- Dashboard deployment
- Staff training

**Budget:** R512,000 (hardware + development)

### 6.2 Phase 2: LoRa Extension (Months 7-12)

**Month 7-8: Gateway Infrastructure**
- Install LoRa gateways at 5 major depots
- Configure gateway-to-network bridges
- Test LoRa-cellular handover

**Month 9-10: Module Deployment**
- Add LoRa modules to 30 depot locomotives
- Validate cost savings from free communication
- Optimize duty cycle management

**Month 11-12: Integration**
- Full system integration testing
- Performance optimization
- Documentation update

**Budget:** R150,000 (gateways + modules)

### 6.3 Phase 3: Satellite Extension (Months 13-18)

**Month 13-14: Satellite Testing**
- Procure 5 Swarm modules for testing
- Validate coverage on Iron Ore line
- Assess latency impact

**Month 15-16: Selective Deployment**
- Install satellite modules on remote route locomotives
- Configure multi-path communication
- Test failover scenarios

**Month 17-18: Optimization**
- Fine-tune module selection algorithms
- Complete coverage analysis
- Final system acceptance

**Budget:** R75,000 (satellite modules + service)

---

## 7. Testing & Validation

### 7.1 Hardware Testing

```cpp
// Self-test routine for production units
bool performSelfTest() {
    Serial.println("Starting self-test...");
    
    // Test cellular modem
    if (!testCellularModem()) {
        Serial.println("ERROR: Cellular modem failed");
        return false;
    }
    
    // Test GPS
    if (!testGPS()) {
        Serial.println("ERROR: GPS failed");
        return false;
    }
    
    // Test SD card
    if (!testSDCard()) {
        Serial.println("ERROR: SD card failed");
        return false;
    }
    
    // Test optional modules
    if (lora && !lora->selfTest()) {
        Serial.println("WARNING: LoRa module issues");
    }
    
    if (satellite && !satellite->selfTest()) {
        Serial.println("WARNING: Satellite module issues");
    }
    
    // Test private APN connection
    if (!testAPNConnection()) {
        Serial.println("ERROR: Cannot connect to private APN");
        return false;
    }
    
    Serial.println("Self-test PASSED");
    return true;
}
```

### 7.2 Network Testing

**Private APN Validation:**
- Verify tunnel establishment
- Test internal server accessibility
- Validate firewall rules
- Confirm data routing

**Coverage Testing:**
- Main routes cellular signal mapping
- LoRa gateway range verification
- Dead zone identification
- Failover scenario validation

---

## 8. Security Considerations

### 8.1 Network Security

**Private APN Benefits:**
- Isolated from public internet
- Direct tunnel to company network
- No exposure to external attacks
- Centralized access control

**Implementation:**
```cpp
// TLS Configuration for MQTT
const char* CA_CERT = R"(
-----BEGIN CERTIFICATE-----
[Company CA Certificate]
-----END CERTIFICATE-----
)";

WiFiClientSecure secureClient;
secureClient.setCACert(CA_CERT);
mqttClient.setClient(secureClient);
```

### 8.2 Device Security

**Security Measures:**
- Unique device certificates
- Encrypted storage on SD card
- Secure boot implementation
- OTA signature verification
- Physical tamper detection

---

## 9. Operational Procedures

### 9.1 Installation Process

1. **Pre-Installation:**
   - Register device IMEI with private APN
   - Configure device ID in database
   - Assign to locomotive

2. **Physical Installation:**
   - Mount in electrical cabinet
   - Connect to 110V locomotive power
   - Install external antenna on roof
   - Secure all connections

3. **Commissioning:**
   - Power on and run self-test
   - Verify GPS lock
   - Confirm private APN connection
   - Test data flow to SQL Server
   - Document installation

### 9.2 Maintenance Procedures

**Regular Maintenance:**
- Monthly: Check dashboard for all units reporting
- Quarterly: Verify antenna connections
- Annually: Replace backup battery
- As needed: Add/swap communication modules

**Troubleshooting Guide:**
| Issue | Check | Solution |
|-------|-------|----------|
| No GPS | Antenna connection | Reconnect/replace antenna |
| No cellular | APN configuration | Verify SIM and APN settings |
| No data | Private network | Check VPN tunnel status |
| Module not detected | Connections | Reseat module connectors |

---

## 10. Cost Analysis

### 10.1 Total Cost Breakdown (100 Units)

**Initial Investment:**
| Item | Cost |
|------|------|
| Base hardware (100 units) | R312,000 |
| Development & integration | R150,000 |
| Private APN setup | R20,000 |
| Installation | R50,000 |
| **Total CAPEX** | **R532,000** |

**Monthly Operating Costs:**
| Service | Units | Cost/Unit | Total |
|---------|-------|-----------|-------|
| Private APN data | 100 | R50 | R5,000 |
| Satellite (selective) | 5 | R75 | R375 |
| **Total OPEX** | | | **R5,375/month** |

### 10.2 ROI Analysis

**Cost Savings:**
- Improved asset utilization: R100,000/year
- Reduced fuel through optimization: R150,000/year
- Prevention of unauthorized use: R50,000/year
- Maintenance optimization: R75,000/year
- **Total savings: R375,000/year**

**Payback Period:** 1.4 years

---

## 11. Future Enhancements

### 11.1 Planned Upgrades

**Year 2:**
- AI-based predictive maintenance
- Integration with wagon tracking
- Driver behavior monitoring
- Fuel consumption tracking

**Year 3:**
- 5G module development
- Edge computing capabilities
- Blockchain-based audit trail
- Advanced analytics dashboard

### 11.2 Scalability

**System Capacity:**
- Current: 100 locomotives
- Expandable to: 1,000+ units
- Database: Petabyte-scale ready
- Dashboard: Real-time for 500+ units

---

## Conclusion

This implementation provides a practical, cost-effective solution for railway locomotive tracking that:

1. **Delivers immediate value** through cellular connectivity
2. **Maintains security** via private APN to company network
3. **Offers flexibility** to add modules where needed
4. **Optimizes costs** through selective technology deployment
5. **Ensures reliability** with proven hardware platform

The phased approach allows for controlled deployment, risk mitigation, and continuous optimization based on operational data.

**Key Success Factors:**
- Private APN configuration for secure connectivity
- Modular design for future expansion
- Store-and-forward for reliability
- Comprehensive testing procedures
- Clear operational procedures

**Next Steps:**
1. Finalize private APN agreement with network provider
2. Order development hardware for prototype
3. Setup test environment with SQL Server
4. Begin firmware development
5. Plan pilot deployment routes

---

*This implementation guide will be continuously updated as the project progresses through each phase.*