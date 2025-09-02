# Backend Services

.NET Core backend services for the locomotive tracking system, providing APIs, data processing, and real-time communications.

## Architecture

The backend is built using microservices architecture with:

- **Web API** - RESTful API for data access and device management
- **MQTT Services** - Real-time data ingestion from tracking devices  
- **Database Layer** - MS SQL Server with spatial data extensions
- **Background Services** - Data processing and maintenance tasks
- **SignalR Hub** - Real-time dashboard updates

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
    Configuration NVARCHAR(MAX) -- JSON configuration
);

-- Location tracking history
CREATE TABLE LocationHistory (
    ID BIGINT IDENTITY PRIMARY KEY,
    DeviceID INT FOREIGN KEY REFERENCES Devices(DeviceID),
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
    INDEX IX_Location_Device_Time (DeviceID, Timestamp) INCLUDE (Position)
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
```

## MQTT Integration

### Topics Structure
```
railway/telemetry/{deviceId}/position    # Position updates
railway/telemetry/{deviceId}/status      # Device status
railway/commands/{deviceId}/config       # Configuration commands
railway/events/{deviceId}/alerts         # Emergency alerts
railway/system/heartbeat                 # System health monitoring
```

### Message Format
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
    }
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
- **Function**: Process incoming telemetry data
- **Frequency**: Real-time (message-driven)
- **Tasks**: Data validation, transformation, storage, alerting

### Maintenance Service  
- **Function**: System maintenance and cleanup
- **Frequency**: Daily at 02:00
- **Tasks**: Archive old data, optimize database, health checks

### Notification Service
- **Function**: Send alerts and notifications
- **Frequency**: Event-driven
- **Tasks**: Geofence violations, device offline alerts, system notifications

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
- **Real-time Map Updates**: Live locomotive positions
- **Alert Notifications**: Immediate system alerts
- **Status Monitoring**: Device connectivity and health
- **Performance Metrics**: System performance dashboards

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

*For API documentation, see [OpenAPI/Swagger](http://localhost:5000/swagger). For deployment guides, see the [Deployment Documentation](../docs/deployment/).*