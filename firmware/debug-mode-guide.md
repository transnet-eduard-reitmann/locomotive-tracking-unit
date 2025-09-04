# Debug Mode Guide - WiFi Access Point Data Extraction

## Overview

The locomotive tracking system includes a comprehensive debug mode that creates a WiFi Access Point for field data extraction and system diagnostics. This feature enables technicians to access on-board data without physical connections, supporting both manual browser access and automated Python scripts.

## Quick Start Guide

### Activation
1. **Hardware Method**: Hold OK+Cancel buttons for 3 seconds
2. **Menu Method**: Navigate to Main Menu → Debug Mode → Activate

### Connection
- **SSID**: `LOCO-{DEVICE_ID}-DEBUG` (e.g., "LOCO-001234-DEBUG")  
- **Password**: `{DEVICE_ID}{YYYYMMDD}` (daily rotation, e.g., "00123420250904")
- **Access**: Open browser to `http://192.168.4.1`

### Session Limits  
- **Timeout**: 30 minutes inactivity, 2 hours maximum
- **Clients**: Maximum 4 concurrent connections

## Debug Mode Architecture

### WiFi Access Point Configuration
```cpp
// WiFi AP Settings
SSID Format: "LOCO-{DEVICE_ID}-DEBUG"
Password Format: "{DEVICE_ID}{YYYYMMDD}"
IP Address: 192.168.4.1
Gateway: 192.168.4.1
Subnet Mask: 255.255.255.0
Channel: 6 (2.4GHz)
Max Clients: 4
Security: WPA2-PSK
```

### Web Server Interface
The debug mode provides a comprehensive web interface accessible via browser:

#### Landing Page (/)
- Device information and system status
- Storage statistics and available space  
- Active session information
- Quick links to major sections

#### File Browser (/files)
- Hierarchical SD card directory navigation
- Individual file download capabilities
- Bulk selection and ZIP archive creation
- File size and timestamp information

#### Log Viewer (/logs)  
- Real-time log streaming via WebSocket
- Historical log file access by category
- Search and filter capabilities
- Export options for selected logs

#### System Status (/status)
- Live GPS positioning data
- Cellular connection status and signal strength
- Train assignment status and history
- Battery level and power consumption
- Temperature and sensor readings

#### Configuration Viewer (/config)
- Current device configuration (read-only)
- GPIO pin assignments and expander status
- Route profiles and communication settings
- Hardware component status

### REST API Endpoints

The debug mode exposes a comprehensive REST API for programmatic access:

#### System Information
```bash
GET /api/system/info          # Device info, uptime, firmware version
GET /api/system/status        # Real-time system status
GET /api/system/storage       # SD card usage and health
GET /api/system/sensors       # Temperature, battery, GPIO states
```

#### File Operations  
```bash
GET /api/files                # List all files and directories
GET /api/files?path={path}    # List contents of specific directory
GET /api/files/download?path={path}  # Download specific file
POST /api/files/archive       # Create ZIP archive of selected files
GET /api/files/tree           # Complete directory tree structure
```

#### Log Access
```bash  
GET /api/logs                 # Available log files and categories
GET /api/logs/{type}          # Specific log type (system, gps, cellular, train)
GET /api/logs/stream          # WebSocket endpoint for real-time streaming
GET /api/logs/search?q={query} # Search logs for specific content
```

#### Configuration Access
```bash
GET /api/config               # Complete device configuration
GET /api/config/routes        # Route profiles and boundaries  
GET /api/config/gpio          # GPIO pin assignments
GET /api/config/cellular      # Cellular/APN configuration (sanitized)
```

## Data Organization

### SD Card File Structure
```
/data/
├── logs/
│   ├── system/          # System logs by date (YYYY-MM-DD.log)
│   │   ├── 2025-09-01.log
│   │   ├── 2025-09-02.log
│   │   └── 2025-09-03.log
│   ├── gps/             # GPS tracking and positioning logs
│   │   ├── positions_2025-09-01.csv
│   │   └── satellites_2025-09-01.log  
│   ├── cellular/        # Cellular connection and network logs
│   │   ├── connection_2025-09-01.log
│   │   └── signal_strength_2025-09-01.csv
│   ├── train/           # Train assignment history and workflow
│   │   ├── assignments_2025-09.log
│   │   └── confirmations_2025-09.log
│   └── debug/           # Debug session logs and access history
│       ├── sessions_2025-09.log
│       └── file_access_2025-09.log
├── config/
│   ├── device.json      # Device configuration backup
│   ├── routes.json      # Route profile data and boundaries
│   ├── gpio_state.json  # GPIO and expander configuration
│   ├── cellular.json    # Cellular/APN settings (passwords removed)
│   └── calibration.json # Sensor calibration and offset data
├── exports/
│   ├── daily/           # Daily data exports and summaries
│   │   ├── summary_2025-09-01.json
│   │   └── positions_2025-09-01.gpx
│   └── sessions/        # Debug session data exports
│       └── session_20250901_143022.zip
└── temp/
    └── debug_archives/  # Temporary ZIP files for bulk downloads
```

## Python API Integration

### Basic Connection Example
```python
import requests
import json
from datetime import datetime

# Connect to debug device
base_url = "http://192.168.4.1/api"

# Get device information
try:
    response = requests.get(f"{base_url}/system/info", timeout=10)
    device_info = response.json()
    
    print(f"Device ID: {device_info['device_id']}")
    print(f"Firmware: {device_info['firmware_version']}")
    print(f"Uptime: {device_info['uptime_hours']} hours")
    print(f"Battery: {device_info['battery_percent']}%")
    
except requests.exceptions.RequestException as e:
    print(f"Connection failed: {e}")
```

### Automated Log Download
```python
import requests
import os
from datetime import datetime, timedelta

def download_recent_logs(device_ip, days_back=7, output_dir="logs"):
    """Download logs from the last N days"""
    base_url = f"http://{device_ip}/api"
    
    # Create output directory
    os.makedirs(output_dir, exist_ok=True)
    
    # Get available logs
    response = requests.get(f"{base_url}/logs")
    available_logs = response.json()
    
    # Filter for recent logs
    cutoff_date = datetime.now() - timedelta(days=days_back)
    
    for log_category in available_logs['categories']:
        print(f"Downloading {log_category} logs...")
        
        # Get logs for this category
        log_response = requests.get(f"{base_url}/logs/{log_category}")
        log_files = log_response.json()
        
        for log_file in log_files['files']:
            log_date = datetime.fromisoformat(log_file['date'])
            
            if log_date >= cutoff_date:
                # Download the log file
                file_response = requests.get(f"{base_url}/files/download", 
                                           params={'path': log_file['path']})
                
                filename = os.path.join(output_dir, f"{log_category}_{log_file['name']}")
                with open(filename, 'wb') as f:
                    f.write(file_response.content)
                    
                print(f"  Downloaded: {filename}")

# Usage
download_recent_logs("192.168.4.1", days_back=30)
```

### Bulk Data Archive Creation
```python
def create_complete_backup(device_ip, output_file="locomotive_backup.zip"):
    """Create complete backup of all device data"""
    base_url = f"http://{device_ip}/api"
    
    # Request creation of complete archive
    archive_request = {
        "paths": [
            "/data/logs/",
            "/data/config/", 
            "/data/exports/"
        ],
        "archive_name": "complete_backup",
        "include_system_info": True
    }
    
    print("Creating archive on device...")
    response = requests.post(f"{base_url}/files/archive", json=archive_request)
    archive_info = response.json()
    
    if archive_info['success']:
        # Download the created archive
        print(f"Downloading archive ({archive_info['size_mb']} MB)...")
        download_url = f"{base_url}/files/download?path={archive_info['path']}"
        
        archive_response = requests.get(download_url, stream=True)
        
        with open(output_file, 'wb') as f:
            for chunk in archive_response.iter_content(chunk_size=8192):
                f.write(chunk)
                
        print(f"Backup completed: {output_file}")
        
        # Clean up temporary file on device
        cleanup_response = requests.delete(f"{base_url}/files/temp/{archive_info['filename']}")
        
    else:
        print(f"Archive creation failed: {archive_info['error']}")
```

## Security and Compliance

### Access Controls
- **Daily Password Rotation**: Passwords change automatically at midnight
- **Session Timeouts**: 30-minute inactivity timeout, 2-hour maximum session
- **Connection Limits**: Maximum 4 concurrent client connections
- **IP Logging**: All client connections logged with IP addresses and MAC (when available)

### Audit Trail
All debug sessions are comprehensively logged:
- Session start/end timestamps
- Client IP addresses and connection duration
- Files accessed and data volume transferred
- API endpoints accessed and query parameters
- Session termination reason (manual, timeout, power, error)

### Data Protection
- **Read-Only Access**: No configuration modification possible via debug interface
- **Sensitive Data Filtering**: Passwords, keys, and certificates excluded from access
- **Secure File Cleanup**: Temporary files automatically cleaned after session
- **Access Summary**: Post-session summary displayed on device UI

## Field Usage Procedures

### Technician Workflow
1. **Safety Check**: Ensure locomotive is stationary and system is stable
2. **Activation**: Use OK+Cancel button combination (3-second hold)
3. **Credential Check**: Note WiFi SSID and password from device display
4. **Connection**: Connect field laptop to debug WiFi network
5. **Data Extraction**: Use browser interface or run Python automation scripts
6. **Documentation**: Record session purpose and data extracted
7. **Clean Exit**: Press EXIT button on device or wait for timeout
8. **Verification**: Confirm normal operation resumed

### Common Data Extraction Scenarios

#### Routine Maintenance Data Collection
- Monthly GPS accuracy analysis
- Cellular signal strength trends
- Battery performance monitoring
- System error frequency analysis

#### Troubleshooting Support
- Real-time log streaming during issues
- Historical data for intermittent problems
- Configuration verification and backup
- Performance metrics for optimization

#### Compliance and Auditing  
- Complete train assignment history
- System access logs and user activity
- Configuration change tracking
- Operational data for regulatory reporting

## Troubleshooting

### Common Issues

#### WiFi AP Not Visible
**Symptoms**: Debug WiFi network doesn't appear in available networks
**Solutions**:
- Verify debug mode activation (check system LED pattern)
- Ensure OK+Cancel buttons held for full 3 seconds
- Check for WiFi interference on channel 6
- Restart debug mode by pressing EXIT and reactivating

#### Cannot Connect to WiFi Network  
**Symptoms**: Device shows correct network but connection fails
**Solutions**:
- Confirm password format: {DEVICE_ID}{YYYYMMDD}
- Check device limit (maximum 4 clients)
- Verify laptop WiFi supports 2.4GHz networks
- Try forgetting and reconnecting to network

#### Web Interface Won't Load
**Symptoms**: Connected to WiFi but browser shows connection errors
**Solutions**:
- Navigate to http://192.168.4.1 (not https)
- Disable VPN and proxy settings
- Try different browser or incognito/private mode
- Check Windows/Mac firewall settings

#### File Downloads Failing
**Symptoms**: Downloads start but fail or corrupt
**Solutions**:
- Verify sufficient disk space on laptop
- Check session timeout (complete downloads within 30 minutes)
- Try smaller file selections for bulk downloads
- Monitor signal strength and move closer if needed

#### Debug Mode Won't Activate
**Symptoms**: Button press has no effect, no UI response
**Solutions**:
- Check GPIO expander I2C communication
- Verify Enhanced UI functionality (other menus work)
- Review system error logs for hardware conflicts
- Try menu-based activation as alternative

### Diagnostic Commands

During development and testing, use these diagnostic commands:

```bash
# Test debug mode activation
pio test -e debug-mode-activation

# Test WiFi AP functionality
pio test -e debug-wifi-ap

# Test web server endpoints  
pio test -e debug-web-server

# Test file operations and downloads
pio test -e debug-file-operations

# Test security and session management
pio test -e debug-security

# Full integration test
pio test -e debug-mode-integration
```

## Advanced Configuration

### Customizing Debug Mode Behavior

Debug mode behavior can be customized through configuration:

```cpp
// debug_config.h
#define DEBUG_WIFI_CHANNEL 6              // WiFi channel (1-13)
#define DEBUG_SESSION_TIMEOUT_MINUTES 30  // Inactivity timeout
#define DEBUG_MAX_SESSION_HOURS 2         // Maximum session duration
#define DEBUG_MAX_CLIENTS 4               // Concurrent connection limit
#define DEBUG_ARCHIVE_MAX_SIZE_MB 500     // Maximum ZIP archive size
#define DEBUG_BATTERY_THRESHOLD 20        // Minimum battery % for debug mode
```

### Development and Testing

For firmware development, debug mode includes development features:

```cpp
#ifdef DEBUG_BUILD
    #define DEBUG_WIFI_PASSWORD "debug123"     // Fixed password for testing
    #define DEBUG_SESSION_TIMEOUT_MINUTES 0   // No timeout for development
    #define DEBUG_ENABLE_CONFIG_WRITE 1       // Allow configuration modification
#endif
```

## Integration with Backend Systems

Debug mode events are integrated with the backend monitoring system:

- **Session Start/End Events**: Published via MQTT for fleet monitoring
- **Audit Database**: All sessions recorded in backend database  
- **Alert Generation**: Extended sessions or unauthorized access triggers alerts
- **Reporting**: Monthly debug session reports for fleet analysis

## Future Enhancements

Planned debug mode improvements include:

- **Remote Debug Activation**: Backend-initiated debug sessions for critical troubleshooting
- **Selective Data Export**: Technician-configurable data filtering for specific issues
- **Automated Diagnostics**: Built-in diagnostic routines for common problems
- **Video Stream Support**: Camera integration for visual diagnostics
- **Multi-language Support**: Localized interfaces for international deployments

---

*This guide provides complete documentation for the debug mode feature. For API reference documentation, see [debug-api-reference.md](debug-api-reference.md). For field procedures, see [../docs/field-operations/debugging-procedures.md](../docs/field-operations/debugging-procedures.md).*