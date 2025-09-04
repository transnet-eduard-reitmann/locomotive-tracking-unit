# Backend Services

.NET Core backend services for the locomotive tracking system, providing APIs, data processing, and real-time communications.

## Architecture

The backend is built using microservices architecture with:

- **Web API** - RESTful API for data access, device management, and train management
- **Train Management Service** - Complete integration with Transnet ITP, TMS, and VDU systems
- **Integration Gateway** - API connectors for existing Transnet systems with data validation
- **MQTT Services** - Real-time data ingestion from tracking devices and train assignment commands
- **Database Layer** - MS SQL Server with spatial data extensions and train management tables
- **Background Services** - Data processing, maintenance tasks, and train synchronization
- **SignalR Hub** - Real-time dashboard updates and train assignment notifications

## Directory Structure

```
backend/
├── api/               # Web API service
│   ├── Controllers/   # API controllers
│   ├── Models/        # Data models and DTOs
│   ├── Services/      # Business logic services
│   └── Middleware/    # Custom middleware components
├── services/          # Background services
│   ├── DataProcessor/ # Telemetry data processing
│   ├── MqttService/   # MQTT broker integration
│   ├── TrainManagement/ # Train management integration services
│   ├── Integration/   # Transnet system connectors (ITP, TMS, VDU)
│   ├── Maintenance/   # System maintenance tasks
│   └── Notifications/ # Alert and notification service
├── database/          # Data access layer
│   ├── Context/       # Entity Framework context
│   ├── Entities/      # Database entities
│   ├── Migrations/    # EF Core migrations
│   └── Repositories/  # Repository pattern implementation
└── mqtt/              # MQTT broker configuration
    ├── config/        # Mosquitto configuration files
    ├── certificates/  # TLS certificates for secure connections
    └── scripts/       # Broker management scripts
```

## Technology Stack

- **.NET Core 8.0** - Main framework
- **Entity Framework Core** - ORM and database access
- **MS SQL Server** - Primary database with spatial extensions
- **SignalR** - Real-time web communication
- **AutoMapper** - Object-to-object mapping
- **Hangfire** - Background job processing
- **Serilog** - Structured logging
- **xUnit** - Unit testing framework

## API Endpoints

### Device Management
```
GET    /api/devices                 # List all tracking devices
GET    /api/devices/{id}            # Get specific device details
POST   /api/devices                 # Register new device
PUT    /api/devices/{id}            # Update device configuration
DELETE /api/devices/{id}            # Remove device
POST   /api/devices/{id}/commands   # Send commands to device
```

### Location Data
```
GET    /api/locations               # Get location history
GET    /api/locations/latest        # Get latest positions for all devices
GET    /api/locations/{deviceId}    # Get location history for specific device
POST   /api/locations               # Submit location data (from MQTT)
GET    /api/locations/route/{id}    # Get locations within route boundary
```

### Fleet Management
```
GET    /api/fleet/status           # Overall fleet status
GET    /api/fleet/summary          # Fleet summary statistics
GET    /api/fleet/alerts           # Active alerts and notifications
GET    /api/fleet/routes           # Defined route information
POST   /api/fleet/geofence         # Create geofence boundaries
```

### Train Management
```
GET    /api/trains                      # List all trains and assignments
GET    /api/trains/{trainNumber}        # Get specific train details
POST   /api/trains/assign               # Assign train number to locomotive
DELETE /api/trains/{trainNumber}        # Clear train assignment
PUT    /api/trains/{trainNumber}/status # Update train status
GET    /api/trains/assignments          # Get active train assignments
```

### Integration
```
GET    /api/integration/itp/plans       # Get train plans from ITP
GET    /api/integration/tms/vehicles    # Get vehicle lists from TMS
POST   /api/integration/vdu/notify      # Send notification to VDU
GET    /api/integration/sync/status     # Get synchronization status
POST   /api/integration/validate        # Validate cross-system data
```

### Reporting
```
GET    /api/reports/utilization    # Asset utilization reports
GET    /api/reports/coverage       # Network coverage analysis
GET    /api/reports/performance    # System performance metrics
GET    /api/reports/costs          # Communication cost analysis
```

## Database Schema

### Core Tables
```sql
-- Device registration and management
CREATE TABLE Devices (
    DeviceID INT IDENTITY PRIMARY KEY,
    IMEI NVARCHAR(20) UNIQUE NOT NULL,
    LocomotiveNumber NVARCHAR(50) NOT NULL,
    InstallDate DATETIME2 NOT NULL,
    ModulesInstalled NVARCHAR(100),
    IsActive BIT DEFAULT 1,
    LastSeen DATETIME2,
    CurrentRoute NVARCHAR(100),
    CurrentTrainNumber NVARCHAR(50), -- Current assigned train
    Configuration NVARCHAR(MAX), -- JSON configuration
    DebugSessionsEnabled BIT DEFAULT 1, -- Allow debug sessions
    LastDebugSession DATETIME2 -- Most recent debug session
);

-- Train management and assignments
CREATE TABLE TrainAssignments (
    AssignmentID INT IDENTITY PRIMARY KEY,
    TrainNumber NVARCHAR(50) NOT NULL,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    AssignedBy NVARCHAR(100) NOT NULL,
    AssignedAt DATETIME2 NOT NULL,
    ConfirmedAt DATETIME2,
    ClearedAt DATETIME2,
    Status NVARCHAR(20) DEFAULT 'PENDING', -- PENDING, CONFIRMED, ACTIVE, CLEARED
    INDEX IX_Train_Active (TrainNumber, Status) WHERE Status IN ('CONFIRMED', 'ACTIVE')
);

-- Integration with Transnet systems
CREATE TABLE SystemIntegration (
    SyncID INT IDENTITY PRIMARY KEY,
    SystemName NVARCHAR(50) NOT NULL, -- ITP, TMS, VDU
    LastSync DATETIME2,
    SyncStatus NVARCHAR(20) DEFAULT 'OK', -- OK, ERROR, WARNING
    ErrorMessage NVARCHAR(MAX),
    RecordsProcessed INT DEFAULT 0
);

-- Location tracking history
CREATE TABLE LocationHistory (
    ID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    TrainNumber NVARCHAR(50), -- Train number at time of position
    Timestamp DATETIME2 NOT NULL,
    Position GEOGRAPHY NOT NULL,
    Speed FLOAT,
    Heading FLOAT,
    Altitude FLOAT,
    Satellites INT,
    HDOP FLOAT,
    CommunicationMethod NVARCHAR(20),
    SignalStrength INT,
    Battery INT,
    Temperature FLOAT,
    INDEX IX_Location_Device_Time (DeviceID, Timestamp) INCLUDE (Position),
    INDEX IX_Location_Train_Time (TrainNumber, Timestamp) INCLUDE (Position)
);

-- Route definitions and boundaries
CREATE TABLE Routes (
    RouteID INT IDENTITY PRIMARY KEY,
    RouteName NVARCHAR(100) NOT NULL,
    RouteGeometry GEOGRAPHY NOT NULL,
    PreferredComm NVARCHAR(20),
    UpdateInterval INT,
    GeofenceBuffer INT,
    CostOptimization BIT DEFAULT 1
);

-- System events and alerts
CREATE TABLE SystemEvents (
    EventID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    Timestamp DATETIME2 NOT NULL,
    EventType NVARCHAR(50) NOT NULL,
    Severity NVARCHAR(20) NOT NULL,
    Description NVARCHAR(500),
    Details NVARCHAR(MAX),
    Acknowledged BIT DEFAULT 0,
    AcknowledgedBy NVARCHAR(100),
    AcknowledgedAt DATETIME2
);

-- Debug session tracking and auditing
CREATE TABLE DebugSessions (
    SessionID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
    SessionStart DATETIME2 NOT NULL,
    SessionEnd DATETIME2,
    InitiatedBy NVARCHAR(100), -- Technician ID or system
    ClientIP NVARCHAR(45), -- IPv4 or IPv6 address
    ClientMAC NVARCHAR(17), -- MAC address if available
    DataTransferredMB DECIMAL(10,2) DEFAULT 0,
    FilesAccessed INT DEFAULT 0,
    SessionDurationMinutes AS DATEDIFF(MINUTE, SessionStart, SessionEnd),
    TerminationReason NVARCHAR(50), -- 'manual_exit', 'timeout', 'power_low', 'system_error'
    SessionNotes NVARCHAR(500),
    AuditFlags NVARCHAR(100), -- JSON flags for compliance
    CreatedAt DATETIME2 DEFAULT GETDATE()
);
```

## MQTT Integration

### Topics Structure
```
railway/telemetry/{deviceId}/position    # Position updates
railway/telemetry/{deviceId}/status      # Device status
railway/commands/{deviceId}/config       # Configuration commands
railway/commands/{deviceId}/train_assign # Train number assignment
railway/commands/{deviceId}/train_clear  # Train number clear
railway/events/{deviceId}/alerts         # Emergency alerts
railway/events/{deviceId}/train_confirmed # Train assignment confirmed
railway/system/heartbeat                 # System health monitoring
```

### Message Format

#### Position Update Message
```json
{
  "deviceId": "LOCO-001",
  "timestamp": "2025-09-01T10:30:45Z",
  "messageType": "position",
  "data": {
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
    },
    "trainNumber": "T12345", // Current assigned train
    "trainStatus": "confirmed"
  }
}
```

#### Train Assignment Command
```json
{
  "deviceId": "LOCO-001",
  "timestamp": "2025-09-01T10:30:45Z",
  "messageType": "train_assign",
  "data": {
    "trainNumber": "T12345",
    "assignedBy": "TCO-JHB-001",
    "requiresConfirmation": true,
    "timeout": 300
  }
}
```

#### Train Confirmation Response
```json
{
  "deviceId": "LOCO-001",
  "timestamp": "2025-09-01T10:32:15Z",
  "messageType": "train_confirmed",
  "data": {
    "trainNumber": "T12345",
    "status": "confirmed", // confirmed, rejected
    "confirmedBy": "DRIVER-001"
  }
}
```

## Development Setup

### Prerequisites
```bash
# Install .NET 8 SDK
# Install SQL Server LocalDB or full SQL Server
# Install Docker (for local MQTT broker)
```

### Database Setup
```bash
# Create database
dotnet ef database update

# Seed initial data
dotnet run --project api -- --seed-data

# Add spatial support
sqlcmd -Q "ALTER DATABASE LocomotiveTracking SET COMPATIBILITY_LEVEL = 150"
```

### Running Services
```bash
# Start MQTT broker (Docker)
docker run -it -p 1883:1883 -p 9001:9001 eclipse-mosquitto

# Start Web API
cd backend/api
dotnet run

# Start background services
cd backend/services
dotnet run

# Run tests
dotnet test
```

## Configuration

### Application Settings
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=LocomotiveTracking;Trusted_Connection=true",
    "MqttBroker": "mqtt://localhost:1883"
  },
  "JwtSettings": {
    "Secret": "your-secret-key",
    "ExpiryInMinutes": 60
  },
  "MqttSettings": {
    "ClientId": "railway-backend",
    "Username": "railway",
    "Password": "secure-password",
    "KeepAlive": 60,
    "QosLevel": 1
  },
  "IntegrationSettings": {
    "ITPApiUrl": "https://itp.transnet.net/api/v1",
    "TMSApiUrl": "https://tms.transnet.net/api/v2",
    "VDUApiUrl": "https://vdu.transnet.net/api/v1",
    "SyncInterval": 300,
    "MaxRetryAttempts": 3,
    "TimeoutSeconds": 30
  }
}
```

### Environment Variables
```bash
# Production settings
export ASPNETCORE_ENVIRONMENT=Production
export ConnectionStrings__DefaultConnection="production-connection-string"
export MqttSettings__Password="production-mqtt-password"
```

## Background Services

### Data Processing Service
- **Function**: Process incoming telemetry data and train assignments
- **Frequency**: Real-time (message-driven)
- **Tasks**: Data validation, transformation, storage, alerting, train status updates

### Train Management Service
- **Function**: Synchronize with Transnet systems and manage train assignments
- **Frequency**: Every 5 minutes (configurable)
- **Tasks**: ITP train plan sync, TMS vehicle list sync, VDU notification handling

### Integration Service
- **Function**: Maintain connectivity with external Transnet systems
- **Frequency**: Continuous with heartbeat monitoring
- **Tasks**: API health monitoring, data validation, error recovery

### Maintenance Service  
- **Function**: System maintenance and cleanup
- **Frequency**: Daily at 02:00
- **Tasks**: Archive old data, optimize database, health checks, integration logs cleanup

### Notification Service
- **Function**: Send alerts and notifications
- **Frequency**: Event-driven
- **Tasks**: Geofence violations, device offline alerts, train assignment notifications, system notifications

### Debug Session Monitoring Service
- **Function**: Monitor and log device debug mode sessions
- **Frequency**: Real-time event-driven
- **Tasks**: Debug session tracking, audit logging, data access monitoring, session analytics
- **Capabilities**: 
  - Track debug session start/end events
  - Monitor data extraction volumes and file access patterns
  - Generate debug session reports for compliance auditing
  - Alert on unauthorized or extended debug sessions
  - Collect debug session statistics for operational insights

## Real-time Features

### SignalR Hubs
```csharp
// Real-time position updates
public class TrackingHub : Hub
{
    public async Task JoinFleetGroup(string fleetId)
    {
        await Groups.AddToGroupAsync(Context.ConnectionId, $"fleet_{fleetId}");
    }
    
    public async Task SendLocationUpdate(string deviceId, LocationData location)
    {
        await Clients.Group($"fleet_{location.FleetId}")
            .SendAsync("LocationUpdate", deviceId, location);
    }
}
```

### Dashboard Integration
- **Real-time Map Updates**: Live locomotive positions with train identification
- **Train Assignment Monitoring**: Live train assignment status and confirmations
- **Alert Notifications**: Immediate system alerts and train assignment notifications
- **Status Monitoring**: Device connectivity, health, and train assignment status
- **Performance Metrics**: System performance dashboards and integration health

## Security

### Authentication & Authorization
- **JWT Tokens** for API authentication
- **Role-based Access Control** (RBAC)
- **API Key Authentication** for device communication
- **TLS Encryption** for all communications

### Data Protection
- **Encryption at Rest**: Database encryption
- **Encryption in Transit**: TLS 1.3 for all connections
- **Access Logging**: Comprehensive audit trails
- **Input Validation**: Protection against injection attacks

## Monitoring & Logging

### Application Monitoring
- **Health Checks**: Endpoint monitoring
- **Performance Metrics**: Response time, throughput
- **Error Tracking**: Exception monitoring and reporting
- **Resource Monitoring**: CPU, memory, database performance

### Logging Strategy
```csharp
// Structured logging with Serilog
Log.Information("Location update received from {DeviceId} at {Position}", 
    deviceId, position);

Log.Warning("Device {DeviceId} offline for {Duration} minutes", 
    deviceId, offlineDuration);

Log.Error(ex, "Failed to process telemetry data for device {DeviceId}", 
    deviceId);
```

## Deployment

### Container Deployment
```dockerfile
# API Service Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY . .
EXPOSE 80
ENTRYPOINT ["dotnet", "LocomotiveTracking.Api.dll"]
```

### Production Checklist
- [ ] Database connection string configured
- [ ] MQTT broker accessible and secured
- [ ] TLS certificates installed
- [ ] Environment variables set
- [ ] Logging configuration verified
- [ ] Health check endpoints enabled
- [ ] Security headers configured

---

*For API documentation, see [OpenAPI/Swagger](http://localhost:5000/swagger). For technical specifications, see the [Implementation Guide](../docs/implementation-guide.md).*