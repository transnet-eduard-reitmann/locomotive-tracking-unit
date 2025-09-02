# South African Railway Locomotive GPS Tracking System
## Raspberry Pi Zero 2W Implementation Guide

---

## Executive Summary

This document provides comprehensive technical documentation for implementing a GPS tracking system using the **Raspberry Pi Zero 2W** as an alternative to the ESP32-based solution. While offering enhanced capabilities for remote management and OTA updates, this approach requires careful consideration of power consumption and complexity trade-offs.

**Key Differentiator:** Full Linux environment enabling SSH access, remote debugging, and sophisticated update mechanisms at the cost of higher power consumption and complexity.

---

## 1. System Architecture

### 1.1 Hardware Platform

```
┌──────────────────────────────────────────────┐
│          RASPBERRY PI ZERO 2W SYSTEM         │
├──────────────────────────────────────────────┤
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Raspberry Pi Zero 2W                  │ │
│  │  • Quad-core ARM Cortex-A53 @ 1GHz    │ │
│  │  • 512MB LPDDR2 SDRAM                  │ │
│  │  • WiFi 802.11b/g/n + Bluetooth 4.2    │ │
│  │  • MicroSD for OS + Storage            │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  SIM7600G HAT for Raspberry Pi         │ │
│  │  • 4G LTE Cat-4 Cellular               │ │
│  │  • Integrated GNSS (GPS/GLONASS)       │ │
│  │  • UART interface to Pi                 │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Power Management System                │ │
│  │  • UPS HAT with 18650 batteries         │ │
│  │  • Smart charging circuit               │ │
│  │  • Power monitoring via I2C             │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │  Expansion Modules (Optional)           │ │
│  │  • LoRa via SPI (GPIO 8,9,10,11)       │ │
│  │  • Satellite via UART2 (GPIO 14,15)    │ │
│  │  • Sensors via I2C (GPIO 2,3)          │ │
│  └────────────────────────────────────────┘ │
│                                              │
└──────────────────────────────────────────────┘
```

### 1.2 Software Stack

```
┌─────────────────────────────────────┐
│         APPLICATION LAYER           │
│  • Python Tracking Application      │
│  • MQTT Client                      │
│  • GPS Processing                   │
│  • OTA Update Manager               │
├─────────────────────────────────────┤
│         SERVICES LAYER              │
│  • systemd service management       │
│  • Docker containers (optional)     │
│  • OpenVPN/WireGuard               │
│  • SSH Server                       │
├─────────────────────────────────────┤
│         DRIVERS LAYER               │
│  • SIM7600G Cellular Driver         │
│  • GPS GPSD Daemon                  │
│  • SPI/I2C/UART Drivers            │
│  • GPIO Control                     │
├─────────────────────────────────────┤
│         OS LAYER                    │
│  • Raspberry Pi OS Lite (32-bit)    │
│  • Linux Kernel 5.15 LTS           │
│  • Read-only root filesystem        │
│  • Overlay filesystem for writes    │
└─────────────────────────────────────────┘
```

---

## 2. Hardware Configuration

### 2.1 Complete Bill of Materials

| Component | Model | Unit Price (ZAR) | Quantity | Total | Purpose |
|-----------|--------|------------------|----------|--------|----------|
| **SBC** | Raspberry Pi Zero 2W | R950 | 1 | R950 | Main processor |
| **Cellular HAT** | Waveshare SIM7600G HAT | R1,200 | 1 | R1,200 | 4G/GPS module |
| **Power HAT** | UPS HAT with RTC | R450 | 1 | R450 | Power management |
| **Batteries** | 18650 3400mAh x3 | R90 | 3 | R270 | 48hr backup |
| **Storage** | 32GB Industrial SD | R250 | 1 | R250 | OS + data |
| **RTC Battery** | CR2032 | R20 | 1 | R20 | Time keeping |
| **Antennas** | 4G/GPS combo | R200 | 1 | R200 | Signal reception |
| **Enclosure** | IP67 with vents | R400 | 1 | R400 | Environmental |
| **Heatsink** | Aluminum + thermal pad | R50 | 1 | R50 | Cooling |
| **Connectors** | Headers, terminals | R100 | 1 | R100 | Connections |
| **LoRa Module** | SX1276 HAT (optional) | R350 | 1 | R350 | LoRa capability |
| **Total Base** | | | | **R3,890** | Complete unit |
| **Total with LoRa** | | | | **R4,240** | With expansion |

### 2.2 Pin Assignments

```python
# GPIO Pin Configuration for Raspberry Pi Zero 2W
PIN_CONFIG = {
    # UART - SIM7600G Communication
    'CELLULAR_TX': 14,    # GPIO14 (UART0_TXD)
    'CELLULAR_RX': 15,    # GPIO15 (UART0_RXD)
    'CELLULAR_POWER': 4,  # GPIO4 - Power control
    'CELLULAR_RESET': 17, # GPIO17 - Reset pin
    
    # SPI - LoRa Module (if installed)
    'LORA_MOSI': 10,      # GPIO10 (SPI0_MOSI)
    'LORA_MISO': 9,       # GPIO9 (SPI0_MISO)
    'LORA_SCLK': 11,      # GPIO11 (SPI0_SCLK)
    'LORA_CS': 8,         # GPIO8 (SPI0_CE0)
    'LORA_RST': 25,       # GPIO25 - Reset
    'LORA_IRQ': 24,       # GPIO24 - Interrupt
    
    # I2C - Sensors and RTC
    'I2C_SDA': 2,         # GPIO2 (I2C1_SDA)
    'I2C_SCL': 3,         # GPIO3 (I2C1_SCL)
    
    # Power Management
    'POWER_MONITOR': 27,  # GPIO27 - Battery voltage ADC
    'SHUTDOWN_BTN': 26,   # GPIO26 - Shutdown button
    'STATUS_LED': 23,     # GPIO23 - Status indicator
    
    # Satellite Module (optional)
    'SAT_TX': 20,         # GPIO20 - Secondary UART
    'SAT_RX': 21,         # GPIO21 - Secondary UART
    'SAT_ENABLE': 16,     # GPIO16 - Enable pin
}
```

### 2.3 Power System Design

```python
# Power monitoring and management
import smbus
import RPi.GPIO as GPIO

class PowerManager:
    def __init__(self):
        self.bus = smbus.SMBus(1)
        self.ups_address = 0x36  # MAX17043 fuel gauge
        self.setup_gpio()
        
    def get_battery_percentage(self):
        """Read battery level from fuel gauge"""
        data = self.bus.read_i2c_block_data(self.ups_address, 0x04, 2)
        percentage = (data[0] << 8 | data[1]) / 256.0
        return min(100, percentage)
    
    def get_power_status(self):
        """Check if running on battery or main power"""
        return {
            'battery_percent': self.get_battery_percentage(),
            'on_battery': GPIO.input(self.POWER_GOOD_PIN) == 0,
            'charging': GPIO.input(self.CHARGE_PIN) == 1,
            'voltage': self.read_voltage(),
            'current': self.read_current(),
            'time_remaining': self.calculate_runtime()
        }
    
    def calculate_runtime(self):
        """Estimate remaining battery time"""
        capacity_mah = 10200  # 3x 3400mAh
        current_ma = self.read_current()
        if current_ma > 0:
            hours = capacity_mah / current_ma
            return hours * 60  # Return minutes
        return 0
        
    def enter_low_power_mode(self):
        """Reduce power consumption"""
        os.system("echo '1' > /sys/devices/system/cpu/cpu1/online")  # Disable CPU cores
        os.system("echo '1' > /sys/devices/system/cpu/cpu2/online")
        os.system("echo '1' > /sys/devices/system/cpu/cpu3/online")
        os.system("tvservice -o")  # Disable HDMI
        os.system("rfkill block bluetooth")  # Disable Bluetooth
```

---

## 3. Software Implementation

### 3.1 Main Application

```python
#!/usr/bin/env python3
"""
Railway Locomotive Tracking System - Raspberry Pi Implementation
"""

import os
import sys
import time
import json
import threading
import logging
from datetime import datetime
from typing import Dict, Optional

import serial
import paho.mqtt.client as mqtt
import gpsd
import requests
from systemd import journal

# Configure logging to systemd journal
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class LocomotiveTracker:
    def __init__(self, config_file='/etc/tracker/config.json'):
        self.load_config(config_file)
        self.init_hardware()
        self.init_communication()
        self.init_storage()
        self.running = True
        
    def load_config(self, config_file):
        """Load configuration from file"""
        with open(config_file, 'r') as f:
            self.config = json.load(f)
        
        self.device_id = self.config['device_id']
        self.apn = self.config['network']['apn']
        self.mqtt_server = self.config['mqtt']['server']
        self.update_interval = self.config['tracking']['interval']
        
    def init_hardware(self):
        """Initialize hardware components"""
        # GPS initialization
        gpsd.connect()
        
        # Cellular modem initialization
        self.modem = serial.Serial(
            port='/dev/ttyUSB0',
            baudrate=115200,
            timeout=1
        )
        self.setup_cellular_connection()
        
        # Optional modules
        self.init_optional_modules()
        
    def init_optional_modules(self):
        """Detect and initialize optional modules"""
        try:
            import spidev
            from pyLoRa import LoRa
            self.lora = LoRa(spi_channel=0, interrupt_pin=24)
            logger.info("LoRa module detected and initialized")
        except ImportError:
            self.lora = None
            logger.info("LoRa module not available")
            
    def setup_cellular_connection(self):
        """Configure private APN connection"""
        commands = [
            'AT+CGDCONT=1,"IP","{}"'.format(self.apn['name']),
            'AT+CSOCKAUTH=1,1,"{}","{}"'.format(
                self.apn['username'], 
                self.apn['password']
            ),
            'AT+NETOPEN',
            'AT+CGPS=1,1'  # Enable GPS
        ]
        
        for cmd in commands:
            self.modem.write((cmd + '\r\n').encode())
            response = self.modem.read(100)
            logger.debug(f"AT Command: {cmd}, Response: {response}")
            time.sleep(1)
            
    def get_gps_position(self) -> Dict:
        """Get current GPS position"""
        try:
            packet = gpsd.get_current()
            if packet.mode >= 2:
                return {
                    'timestamp': datetime.utcnow().isoformat(),
                    'latitude': packet.lat,
                    'longitude': packet.lon,
                    'altitude': packet.alt if packet.mode >= 3 else None,
                    'speed': packet.hspeed if hasattr(packet, 'hspeed') else 0,
                    'heading': packet.track if hasattr(packet, 'track') else 0,
                    'satellites': packet.sats if hasattr(packet, 'sats') else 0,
                    'hdop': packet.hdop if hasattr(packet, 'hdop') else 99
                }
        except Exception as e:
            logger.error(f"GPS error: {e}")
            return None
            
    def transmit_telemetry(self, data: Dict):
        """Send telemetry via best available method"""
        # Try cellular first
        if self.cellular_connected():
            return self.send_via_mqtt(data)
            
        # Try LoRa if available
        if self.lora and self.lora.gateway_in_range():
            return self.send_via_lora(data)
            
        # Store for later transmission
        self.store_offline(data)
        return False
        
    def send_via_mqtt(self, data: Dict) -> bool:
        """Send data via MQTT over cellular"""
        try:
            topic = f"telemetry/{self.device_id}"
            payload = json.dumps(data)
            result = self.mqtt_client.publish(topic, payload, qos=1)
            return result.rc == mqtt.MQTT_ERR_SUCCESS
        except Exception as e:
            logger.error(f"MQTT transmission error: {e}")
            return False
            
    def check_for_updates(self):
        """Check and apply OTA updates"""
        try:
            # Check for system updates
            os.system('sudo apt-get update -qq')
            updates = os.popen('apt list --upgradable 2>/dev/null | grep -c upgradable').read()
            
            if int(updates.strip()) > 0:
                logger.info("System updates available, installing...")
                os.system('sudo apt-get upgrade -y -qq')
                
            # Check for application updates
            response = requests.get(
                f"{self.config['ota']['server']}/version",
                timeout=10
            )
            
            if response.status_code == 200:
                latest_version = response.json()['version']
                if latest_version > self.config['version']:
                    self.perform_application_update(latest_version)
                    
        except Exception as e:
            logger.error(f"Update check failed: {e}")
            
    def perform_application_update(self, version):
        """Download and install application update"""
        try:
            # Download update
            url = f"{self.config['ota']['server']}/download/{version}"
            response = requests.get(url, stream=True)
            
            update_file = f"/tmp/tracker_{version}.tar.gz"
            with open(update_file, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    f.write(chunk)
                    
            # Verify checksum
            if self.verify_update_integrity(update_file):
                # Install update
                os.system(f'tar -xzf {update_file} -C /opt/tracker/')
                os.system('sudo systemctl restart tracker')
                logger.info(f"Successfully updated to version {version}")
            else:
                logger.error("Update verification failed")
                
        except Exception as e:
            logger.error(f"Application update failed: {e}")
            
    def run(self):
        """Main tracking loop"""
        logger.info(f"Locomotive Tracker v{self.config['version']} started")
        
        # Start update checker thread
        update_thread = threading.Thread(target=self.update_checker_thread)
        update_thread.daemon = True
        update_thread.start()
        
        while self.running:
            try:
                # Get GPS position
                position = self.get_gps_position()
                
                if position:
                    # Add system telemetry
                    position['battery'] = self.power_manager.get_battery_percentage()
                    position['signal_strength'] = self.get_signal_strength()
                    position['device_id'] = self.device_id
                    
                    # Transmit telemetry
                    if self.transmit_telemetry(position):
                        logger.debug(f"Telemetry sent: {position}")
                    else:
                        logger.warning("Telemetry transmission failed, stored locally")
                        
                # Adaptive sleep based on movement
                sleep_time = self.calculate_sleep_interval(
                    position['speed'] if position else 0
                )
                time.sleep(sleep_time)
                
            except KeyboardInterrupt:
                logger.info("Shutdown requested")
                self.running = False
            except Exception as e:
                logger.error(f"Main loop error: {e}")
                time.sleep(10)
                
        self.cleanup()
        
    def cleanup(self):
        """Clean shutdown"""
        self.mqtt_client.disconnect()
        self.modem.close()
        logger.info("Tracker shutdown complete")
        
if __name__ == "__main__":
    tracker = LocomotiveTracker()
    tracker.run()
```

### 3.2 System Service Configuration

```ini
# /etc/systemd/system/tracker.service
[Unit]
Description=Railway Locomotive Tracker
After=network.target gpsd.service
Wants=gpsd.service

[Service]
Type=simple
User=pi
Group=pi
WorkingDirectory=/opt/tracker
ExecStart=/usr/bin/python3 /opt/tracker/tracker.py
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

# Security hardening
PrivateTmp=yes
NoNewPrivileges=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/tracker /var/log/tracker

# Resource limits
CPUQuota=80%
MemoryMax=256M
TasksMax=10

# Watchdog
WatchdogSec=60
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
```

### 3.3 OTA Update System

```python
# /opt/tracker/ota_manager.py
import os
import subprocess
import hashlib
import json
import requests
from typing import Dict, Optional
import logging

logger = logging.getLogger(__name__)

class OTAManager:
    def __init__(self, config: Dict):
        self.config = config
        self.update_server = config['ota']['server']
        self.current_version = config['version']
        
    def check_for_updates(self) -> Optional[Dict]:
        """Check if updates are available"""
        try:
            # System updates via apt
            system_updates = self.check_system_updates()
            
            # Application updates
            app_updates = self.check_application_updates()
            
            # Firmware updates (if applicable)
            firmware_updates = self.check_firmware_updates()
            
            return {
                'system': system_updates,
                'application': app_updates,
                'firmware': firmware_updates
            }
        except Exception as e:
            logger.error(f"Update check failed: {e}")
            return None
            
    def check_system_updates(self) -> bool:
        """Check for OS updates"""
        result = subprocess.run(
            ['apt', 'list', '--upgradable'],
            capture_output=True,
            text=True
        )
        return 'upgradable' in result.stdout
        
    def check_application_updates(self) -> Optional[str]:
        """Check for application updates"""
        try:
            response = requests.get(
                f"{self.update_server}/api/version",
                headers={'Device-ID': self.config['device_id']},
                timeout=30
            )
            
            if response.status_code == 200:
                data = response.json()
                if data['version'] > self.current_version:
                    return data['version']
        except Exception as e:
            logger.error(f"Application update check failed: {e}")
        return None
        
    def perform_update(self, update_type: str, version: str = None):
        """Perform the actual update"""
        if update_type == 'system':
            return self.update_system()
        elif update_type == 'application':
            return self.update_application(version)
        elif update_type == 'firmware':
            return self.update_firmware(version)
            
    def update_system(self) -> bool:
        """Update OS packages"""
        try:
            # Update package list
            subprocess.run(['sudo', 'apt-get', 'update'], check=True)
            
            # Upgrade packages
            subprocess.run(
                ['sudo', 'apt-get', 'upgrade', '-y'],
                check=True,
                timeout=600
            )
            
            # Clean up
            subprocess.run(['sudo', 'apt-get', 'autoremove', '-y'])
            subprocess.run(['sudo', 'apt-get', 'autoclean'])
            
            logger.info("System update completed successfully")
            return True
            
        except subprocess.CalledProcessError as e:
            logger.error(f"System update failed: {e}")
            return False
            
    def update_application(self, version: str) -> bool:
        """Update the tracking application"""
        try:
            # Download update package
            update_url = f"{self.update_server}/api/download/{version}"
            response = requests.get(update_url, stream=True)
            
            update_file = f"/tmp/update_{version}.tar.gz"
            sha256_hash = hashlib.sha256()
            
            with open(update_file, 'wb') as f:
                for chunk in response.iter_content(chunk_size=8192):
                    sha256_hash.update(chunk)
                    f.write(chunk)
                    
            # Verify checksum
            expected_hash = response.headers.get('X-Checksum-SHA256')
            if sha256_hash.hexdigest() != expected_hash:
                raise ValueError("Checksum verification failed")
                
            # Backup current version
            subprocess.run([
                'tar', '-czf', 
                f'/opt/tracker/backup_{self.current_version}.tar.gz',
                '/opt/tracker'
            ])
            
            # Extract update
            subprocess.run([
                'tar', '-xzf', update_file,
                '-C', '/opt/tracker/',
                '--strip-components=1'
            ], check=True)
            
            # Update config with new version
            self.config['version'] = version
            with open('/etc/tracker/config.json', 'w') as f:
                json.dump(self.config, f, indent=2)
                
            # Restart service
            subprocess.run(['sudo', 'systemctl', 'restart', 'tracker'], check=True)
            
            logger.info(f"Application updated to version {version}")
            return True
            
        except Exception as e:
            logger.error(f"Application update failed: {e}")
            self.rollback_application()
            return False
            
    def rollback_application(self):
        """Rollback to previous version"""
        try:
            backup_file = f'/opt/tracker/backup_{self.current_version}.tar.gz'
            if os.path.exists(backup_file):
                subprocess.run([
                    'tar', '-xzf', backup_file,
                    '-C', '/',
                    '--overwrite'
                ], check=True)
                
                subprocess.run(['sudo', 'systemctl', 'restart', 'tracker'])
                logger.info("Rollback completed")
        except Exception as e:
            logger.error(f"Rollback failed: {e}")
```

---

## 4. Network Configuration

### 4.1 Private APN Setup

```bash
#!/bin/bash
# /etc/tracker/setup_apn.sh

# Configure NetworkManager for private APN
nmcli connection add type gsm \
    con-name "railway-apn" \
    ifname cdc-wdm0 \
    gsm.apn "transnet.m2m" \
    gsm.username "railway" \
    gsm.password "secure_password" \
    connection.autoconnect yes \
    connection.autoconnect-priority 100

# Configure routing for private network
cat > /etc/NetworkManager/dispatcher.d/30-private-route << 'EOF'
#!/bin/bash
if [ "$1" = "cdc-wdm0" ] && [ "$2" = "up" ]; then
    # Add route to private network
    ip route add 10.50.0.0/16 dev cdc-wdm0
    # Set DNS for private network
    echo "nameserver 10.50.1.1" > /etc/resolv.conf
fi
EOF

chmod +x /etc/NetworkManager/dispatcher.d/30-private-route
```

### 4.2 Remote Access Configuration

```python
# /opt/tracker/remote_access.py
import subprocess
import os

class RemoteAccessManager:
    def __init__(self):
        self.vpn_config = '/etc/wireguard/wg0.conf'
        self.ssh_config = '/etc/ssh/sshd_config'
        
    def setup_wireguard_vpn(self):
        """Configure WireGuard for secure remote access"""
        config = """
[Interface]
PrivateKey = {private_key}
Address = 10.100.0.{device_id}/24
ListenPort = 51820

[Peer]
PublicKey = {server_public_key}
Endpoint = 10.50.100.1:51820
AllowedIPs = 10.50.0.0/16
PersistentKeepalive = 25
"""
        
        # Generate keys
        private_key = subprocess.check_output(
            ['wg', 'genkey'],
            text=True
        ).strip()
        
        # Write config
        with open(self.vpn_config, 'w') as f:
            f.write(config.format(
                private_key=private_key,
                device_id=self.get_device_id(),
                server_public_key=self.get_server_public_key()
            ))
            
        # Enable VPN
        subprocess.run(['sudo', 'systemctl', 'enable', 'wg-quick@wg0'])
        subprocess.run(['sudo', 'systemctl', 'start', 'wg-quick@wg0'])
        
    def secure_ssh_config(self):
        """Harden SSH configuration"""
        ssh_settings = """
Port 22222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2
AllowUsers pi
"""
        
        with open(self.ssh_config, 'a') as f:
            f.write(ssh_settings)
            
        subprocess.run(['sudo', 'systemctl', 'restart', 'ssh'])
```

---

## 5. Database Integration

### 5.1 MS SQL Server Connection

```python
# /opt/tracker/database.py
import pyodbc
import json
from datetime import datetime
from typing import Dict, List
import logging

logger = logging.getLogger(__name__)

class DatabaseManager:
    def __init__(self, config: Dict):
        self.config = config['database']
        self.connection = None
        self.connect()
        
    def connect(self):
        """Establish connection to MS SQL Server"""
        try:
            connection_string = (
                f"DRIVER={{ODBC Driver 17 for SQL Server}};"
                f"SERVER={self.config['server']};"
                f"DATABASE={self.config['database']};"
                f"UID={self.config['username']};"
                f"PWD={self.config['password']};"
                f"Encrypt=yes;"
                f"TrustServerCertificate=yes;"
            )
            
            self.connection = pyodbc.connect(connection_string)
            logger.info("Database connection established")
            
        except Exception as e:
            logger.error(f"Database connection failed: {e}")
            
    def insert_telemetry(self, data: Dict):
        """Insert telemetry data into database"""
        try:
            cursor = self.connection.cursor()
            
            # Convert to SQL Server geography type
            point = f"POINT({data['longitude']} {data['latitude']})"
            
            query = """
                INSERT INTO LocationHistory 
                (DeviceID, Timestamp, Position, Speed, Heading, 
                 Altitude, Satellites, HDOP, Battery, Temperature,
                 CommunicationMethod, SignalStrength)
                VALUES (?, ?, geography::STPointFromText(?, 4326), 
                        ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """
            
            cursor.execute(query, 
                self.config['device_id'],
                datetime.utcnow(),
                point,
                data.get('speed', 0),
                data.get('heading', 0),
                data.get('altitude', 0),
                data.get('satellites', 0),
                data.get('hdop', 99),
                data.get('battery', 0),
                data.get('temperature', 0),
                data.get('comm_method', 'Cellular'),
                data.get('signal_strength', 0)
            )
            
            self.connection.commit()
            
        except Exception as e:
            logger.error(f"Database insert failed: {e}")
            self.connection.rollback()
```

---

## 6. Testing & Deployment

### 6.1 Unit Testing

```python
# /tests/test_tracker.py
import unittest
from unittest.mock import Mock, patch
import sys
sys.path.append('/opt/tracker')

from tracker import LocomotiveTracker

class TestTracker(unittest.TestCase):
    def setUp(self):
        self.tracker = LocomotiveTracker('/tests/test_config.json')
        
    def test_gps_parsing(self):
        """Test GPS data parsing"""
        mock_gps_data = {
            'lat': -26.195246,
            'lon': 28.034088,
            'alt': 1680.5,
            'speed': 45.6
        }
        
        with patch('gpsd.get_current') as mock_gpsd:
            mock_gpsd.return_value = Mock(**mock_gps_data)
            position = self.tracker.get_gps_position()
            
            self.assertIsNotNone(position)
            self.assertEqual(position['latitude'], -26.195246)
            self.assertEqual(position['speed'], 45.6)
            
    def test_mqtt_transmission(self):
        """Test MQTT data transmission"""
        test_data = {
            'latitude': -26.195246,
            'longitude': 28.034088,
            'speed': 45.6
        }
        
        with patch.object(self.tracker.mqtt_client, 'publish') as mock_publish:
            mock_publish.return_value = Mock(rc=0)
            result = self.tracker.send_via_mqtt(test_data)
            
            self.assertTrue(result)
            mock_publish.assert_called_once()
            
    def test_power_monitoring(self):
        """Test power management functions"""
        battery_level = self.tracker.power_manager.get_battery_percentage()
        self.assertIsInstance(battery_level, (int, float))
        self.assertTrue(0 <= battery_level <= 100)
        
if __name__ == '__main__':
    unittest.main()
```

### 6.2 Deployment Script

```bash
#!/bin/bash
# /scripts/deploy.sh

set -e

echo "Railway Tracker Deployment Script"
echo "================================="

# Update system
echo "Updating system packages..."
sudo apt-get update
sudo apt-get upgrade -y

# Install dependencies
echo "Installing dependencies..."
sudo apt-get install -y \
    python3-pip \
    python3-dev \
    gpsd \
    gpsd-clients \
    python3-gps \
    libatlas-base-dev \
    libopenjp2-7 \
    unixodbc-dev \
    git

# Install Python packages
echo "Installing Python packages..."
pip3 install --upgrade pip
pip3 install \
    paho-mqtt \
    pyserial \
    gpsd-py3 \
    requests \
    pyodbc \
    systemd-python \
    RPi.GPIO \
    smbus2 \
    spidev

# Create directories
echo "Creating directories..."
sudo mkdir -p /opt/tracker
sudo mkdir -p /etc/tracker
sudo mkdir -p /var/lib/tracker
sudo mkdir -p /var/log/tracker

# Copy application files
echo "Installing application..."
sudo cp -r /tmp/tracker/* /opt/tracker/
sudo cp /tmp/config.json /etc/tracker/

# Set permissions
sudo chown -R pi:pi /opt/tracker
sudo chown -R pi:pi /var/lib/tracker
sudo chown -R pi:pi /var/log/tracker

# Install systemd service
echo "Installing service..."
sudo cp /opt/tracker/tracker.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable tracker.service

# Configure read-only filesystem
echo "Configuring read-only filesystem..."
sudo raspi-config nonint do_overlayfs 0

# Start service
echo "Starting tracker service..."
sudo systemctl start tracker.service

echo "Deployment complete!"
echo "Check status: sudo systemctl status tracker"
```

---

## 7. Monitoring & Maintenance

### 7.1 Health Monitoring

```python
# /opt/tracker/health_monitor.py
import psutil
import subprocess
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

class HealthMonitor:
    def __init__(self):
        self.thresholds = {
            'cpu_percent': 80,
            'memory_percent': 85,
            'disk_percent': 90,
            'temperature': 70
        }
        
    def get_system_health(self):
        """Get comprehensive system health metrics"""
        return {
            'timestamp': datetime.utcnow().isoformat(),
            'cpu': {
                'percent': psutil.cpu_percent(interval=1),
                'cores': psutil.cpu_count(),
                'frequency': psutil.cpu_freq().current
            },
            'memory': {
                'percent': psutil.virtual_memory().percent,
                'available_mb': psutil.virtual_memory().available / 1024 / 1024,
                'total_mb': psutil.virtual_memory().total / 1024 / 1024
            },
            'disk': {
                'percent': psutil.disk_usage('/').percent,
                'free_gb': psutil.disk_usage('/').free / 1024 / 1024 / 1024
            },
            'temperature': self.get_cpu_temperature(),
            'uptime': self.get_uptime(),
            'processes': len(psutil.pids()),
            'network': self.get_network_stats()
        }
        
    def get_cpu_temperature(self):
        """Read CPU temperature"""
        try:
            temp = subprocess.check_output(
                ['vcgencmd', 'measure_temp'],
                text=True
            )
            return float(temp.split('=')[1].split("'")[0])
        except:
            return 0
            
    def get_uptime(self):
        """Get system uptime in seconds"""
        with open('/proc/uptime', 'r') as f:
            uptime_seconds = float(f.readline().split()[0])
        return uptime_seconds
        
    def check_alerts(self, health_data):
        """Check if any metrics exceed thresholds"""
        alerts = []
        
        if health_data['cpu']['percent'] > self.thresholds['cpu_percent']:
            alerts.append(f"High CPU usage: {health_data['cpu']['percent']}%")
            
        if health_data['memory']['percent'] > self.thresholds['memory_percent']:
            alerts.append(f"High memory usage: {health_data['memory']['percent']}%")
            
        if health_data['temperature'] > self.thresholds['temperature']:
            alerts.append(f"High temperature: {health_data['temperature']}°C")
            
        return alerts
```

---

## 8. Cost Analysis

### 8.1 Total Cost Comparison

| Category | ESP32 Solution | Raspberry Pi Solution | Difference |
|----------|---------------|----------------------|------------|
| **Hardware per unit** | R3,120 | R3,890 | +R770 |
| **With LoRa** | R3,420 | R4,240 | +R820 |
| **100 units base** | R312,000 | R389,000 | +R77,000 |
| **Development** | R150,000 | R200,000 | +R50,000 |
| **Total CAPEX** | R512,000 | R639,000 | +R127,000 |
| **Annual OPEX** | R64,500 | R76,500 | +R12,000 |
| **5-Year TCO** | R854,500 | R1,021,500 | +R167,000 |

### 8.2 Power Consumption Costs

| Metric | ESP32 | Raspberry Pi | Impact |
|--------|-------|--------------|--------|
| **Idle Current** | 80mA | 150mA | 1.9x higher |
| **Active Current** | 200mA | 350mA | 1.75x higher |
| **Daily Energy** | 2.4Wh | 4.2Wh | 1.75x higher |
| **Battery Size Needed** | 6800mAh | 10000mAh | +47% capacity |
| **Battery Cost** | R120 | R270 | +R150/unit |

---

## 9. Advantages & Disadvantages

### 9.1 Raspberry Pi Advantages ✅

**Development & Maintenance:**
- Full SSH access for remote debugging
- Standard Linux tools and utilities
- Easier development with Python/Node.js
- Rich ecosystem of libraries
- Multiple update mechanisms
- Container support (Docker)

**Capabilities:**
- Edge computing possibilities
- Local database capability
- Complex data processing
- Video/image processing potential
- VPN and advanced networking
- Full filesystem access

### 9.2 Raspberry Pi Disadvantages ❌

**Operational Challenges:**
- 30-45 second boot time (vs 1 second)
- SD card corruption risk
- Higher power consumption
- More complex = more failure points
- Requires Linux administration skills
- Higher cost (+25% hardware)

**Reliability Concerns:**
- Filesystem corruption on power loss
- SD card wear-out (1-2 year lifespan)
- Non-deterministic timing
- Memory management issues
- Thermal management required

---

## 10. Final Recommendation

### For Railway Tracking: ESP32 Remains Optimal

While the Raspberry Pi offers superior remote management and flexibility, the **ESP32 with OTA** provides the better solution for railway tracking due to:

1. **Power Efficiency:** Critical for 48-hour battery backup requirement
2. **Reliability:** No filesystem corruption risk, faster recovery
3. **Cost:** R167,000 less over 5 years for 100 units
4. **Simplicity:** Less complexity = higher reliability
5. **Boot Time:** 1 second vs 45 seconds matters for availability

### When Raspberry Pi Makes Sense:

- Need for SSH debugging during development
- Complex edge analytics requirements
- Integration with Linux-specific tools
- Frequent software architecture changes
- Advanced networking requirements

### Recommended Approach:

Use **ESP32 for production fleet** (95 units) with **Raspberry Pi for development/testing** (5 units) to get benefits of both platforms.