# Enhanced UI Specifications - Train Management Interface

## Overview

This document defines the complete technical specifications for the enhanced user interface implementation on the LILYGO T-SIM7600G-H locomotive tracking system. The enhanced UI enables train assignment workflows, system status monitoring, and operator interaction through a 2.8" color TFT display with 4-button navigation.

## Hardware Specifications

### Display System
- **Display**: 2.8" Color TFT LCD
- **Resolution**: 320×240 pixels (QVGA)
- **Color Depth**: 16-bit (65,536 colors)
- **Interface**: SPI (4-wire + control signals)
- **Backlight**: LED backlight with PWM control
- **Touch**: Optional resistive touch (not used in railway environment)
- **Operating Temperature**: -20°C to +70°C
- **Viewing Angle**: 160° horizontal, 140° vertical

### Input System
- **Navigation**: 4-button tactile switch array
- **Button Types**: Momentary contact, normally open
- **Debounce**: Hardware debouncing with RC circuit + software debouncing
- **Button Life**: >1,000,000 actuations
- **Operating Force**: 2.5N ±0.5N
- **Sealing**: IP67 rated sealed switches

### Status Indicators
- **LEDs**: 4× multi-color LEDs (RGB)
- **Positions**: GPS, Cellular, Train Assignment, System Status
- **Control**: Via MCP23017 GPIO expander
- **Current**: 10mA per LED (software controlled)
- **Visibility**: Daylight readable with diffuser lens

### Audio Feedback
- **Buzzer**: Piezoelectric buzzer
- **Frequency Range**: 2-4 kHz
- **Volume**: 75-85 dB @ 10cm
- **Control**: Via GPIO expander with PWM modulation
- **Use Cases**: Train assignment notifications, alerts, button feedback

## Software Architecture

### Display Driver Stack
```cpp
// Display driver hierarchy
┌─────────────────────┐
│   UI Manager        │ ← Application layer
├─────────────────────┤
│   Screen Manager    │ ← Screen switching and lifecycle
├─────────────────────┤
│   Widget System     │ ← UI components and rendering
├─────────────────────┤
│   TFT_eSPI Driver   │ ← Hardware abstraction layer
├─────────────────────┤
│   ESP32 SPI HAL     │ ← Hardware interface
└─────────────────────┘
```

### Input Processing System
```cpp
// Input processing flow
Button Hardware → GPIO Expander → I2C Interface → Button Driver → 
Event Queue → UI Manager → Screen Handler → Action Execution
```

### State Management
```cpp
enum SystemState {
    SYSTEM_BOOT,
    SYSTEM_IDLE,
    SYSTEM_TRACKING,
    TRAIN_ASSIGNMENT_PENDING,
    TRAIN_ASSIGNMENT_CONFIRMED,
    TRAIN_ASSIGNMENT_ACTIVE,
    SYSTEM_ERROR,
    SYSTEM_MAINTENANCE
};

enum DisplayState {
    DISPLAY_OFF,
    DISPLAY_DIM,
    DISPLAY_ACTIVE,
    DISPLAY_SLEEPING
};
```

## Screen Specifications

### Main Status Screen
**Purpose**: Primary operational display showing system and train status
**Layout**: 
```
┌─────────────────────────────────────┐
│ LOCOMOTIVE TRACKER         [GPS][4G] │
│                                     │
│ LOCO: 001234                        │
│ TRAIN: T98765          [CONFIRMED]   │
│                                     │
│ STATUS: TRACKING                    │
│ SPEED: 45 km/h                      │
│ POSITION: 26°11'42"S 28°02'03"E     │
│                                     │
│ LAST UPDATE: 10:30:15               │
│                [OK] [MENU]          │
└─────────────────────────────────────┘
```

**Elements**:
- Header with system status icons
- Locomotive number (device configuration)  
- Current train number and assignment status
- Operational status and speed
- GPS coordinates (abbreviated format)
- Last successful data transmission time
- Navigation hints

### Train Assignment Screen
**Purpose**: Handle train number assignment workflow
**Layout**:
```
┌─────────────────────────────────────┐
│ TRAIN ASSIGNMENT REQUEST            │
│                                     │
│ ASSIGN TRAIN NUMBER:                │
│                                     │
│        T98765                       │
│                                     │
│ ASSIGNED BY: TCO-JHB-001            │
│ TIME: 10:25:30                      │
│                                     │
│ CONFIRM ASSIGNMENT?                 │
│              [YES]    [NO]          │
└─────────────────────────────────────┘
```

**Workflow States**:
1. **Assignment Received**: Display train number and source
2. **Confirmation Pending**: Wait for operator response
3. **Confirmed**: Show confirmation and transition to active
4. **Rejected**: Show rejection reason options
5. **Timeout**: Auto-reject after 5 minutes

### Menu System
**Purpose**: Access device configuration and diagnostic information
**Layout**:
```
┌─────────────────────────────────────┐
│ MAIN MENU                           │
│                                     │
│ > System Information                │
│   Train History                     │
│   Network Status                    │
│   Settings                          │
│   Diagnostics                       │
│                                     │
│              [↑] [↓] [SELECT] [BACK]│
└─────────────────────────────────────┘
```

**Navigation**:
- Up/Down arrows for menu selection
- Select button to enter submenu
- Back button to return to previous screen
- Auto-timeout to main screen after 2 minutes inactivity

### System Information Screen
**Purpose**: Display device and system status details
**Layout**:
```
┌─────────────────────────────────────┐
│ SYSTEM INFORMATION                  │
│                                     │
│ DEVICE ID: LOCO-001                 │
│ FIRMWARE: v2.1.0                    │
│ UPTIME: 15d 8h 23m                  │
│                                     │
│ BATTERY: 85% (38.2V)                │
│ TEMPERATURE: 28.5°C                 │
│                                     │
│ STORAGE: 2.1GB / 32GB               │
│                [BACK]               │
└─────────────────────────────────────┘
```

### Network Status Screen
**Purpose**: Communication system status and diagnostics
**Layout**:
```
┌─────────────────────────────────────┐
│ NETWORK STATUS                      │
│                                     │
│ CELLULAR: CONNECTED                 │
│ PROVIDER: Vodacom 4G               │
│ SIGNAL: -75 dBm (Good)              │
│ DATA SENT: 2.3 MB                  │
│                                     │
│ LORA: STANDBY                       │
│ SATELLITE: NOT INSTALLED            │
│                [BACK]               │
└─────────────────────────────────────┘
```

### Error/Alert Screen
**Purpose**: Display system errors and alerts
**Layout**:
```
┌─────────────────────────────────────┐
│ ⚠ SYSTEM ALERT                     │
│                                     │
│ GPS SIGNAL LOST                     │
│                                     │
│ TIME: 10:45:12                      │
│ LOCATION: Last known position       │
│ 26°11'42"S 28°02'03"E               │
│                                     │
│ ACTION: Check antenna connection    │
│                [ACKNOWLEDGE]        │
└─────────────────────────────────────┘
```

## User Interface Guidelines

### Typography
- **Primary Font**: Liberation Sans 12px (system status, train numbers)
- **Secondary Font**: Liberation Sans 10px (details, timestamps)
- **Monospace Font**: Liberation Mono 10px (coordinates, technical data)
- **Large Font**: Liberation Sans 16px (alerts, confirmations)

### Color Scheme
```cpp
// Primary Colors
#define COLOR_BACKGROUND    0x0000  // Black
#define COLOR_TEXT_PRIMARY  0xFFFF  // White  
#define COLOR_TEXT_SECONDARY 0x8410 // Light gray
#define COLOR_ACCENT        0x07FF  // Cyan

// Status Colors  
#define COLOR_SUCCESS       0x07E0  // Green
#define COLOR_WARNING       0xFFE0  // Yellow
#define COLOR_ERROR         0xF800  // Red
#define COLOR_INFO          0x03EF  // Blue

// Train Assignment Colors
#define COLOR_PENDING       0xFFE0  // Yellow
#define COLOR_CONFIRMED     0x07E0  // Green  
#define COLOR_ACTIVE        0x03EF  // Blue
#define COLOR_REJECTED      0xF800  // Red
```

### Button Behavior
- **Short Press**: Navigate, select, confirm
- **Long Press (1 second)**: Alternative actions, shortcuts
- **Double Press**: Quick actions (back to main screen)
- **Button Feedback**: Visual highlight + optional beep

### Screen Transitions
- **Transition Type**: Slide left/right for screen changes
- **Animation Duration**: 200ms
- **Easing**: Ease-in-out curve
- **Fallback**: Instant transition if performance limited

## Status LED Specifications

### LED Functions and Colors
```cpp
// GPS Status LED
LED_GPS_OFF      // No GPS signal
LED_GPS_RED      // GPS searching  
LED_GPS_YELLOW   // GPS acquiring
LED_GPS_GREEN    // GPS locked (>4 satellites)
LED_GPS_BLUE     // GPS precision mode (>8 satellites)

// Cellular Status LED  
LED_CELL_OFF     // No cellular connection
LED_CELL_RED     // Connection failed/searching
LED_CELL_YELLOW  // Connected, poor signal
LED_CELL_GREEN   // Connected, good signal
LED_CELL_BLUE    // Connected, excellent signal

// Train Assignment Status LED
LED_TRAIN_OFF    // No train assigned
LED_TRAIN_YELLOW // Assignment pending confirmation
LED_TRAIN_GREEN  // Train assigned and confirmed
LED_TRAIN_BLUE   // Train active and tracking
LED_TRAIN_RED    // Assignment error/conflict

// System Status LED
LED_SYS_OFF      // System off
LED_SYS_RED      // System error/fault
LED_SYS_YELLOW   // System warning/limited operation
LED_SYS_GREEN    // System operational
LED_SYS_BLUE     // System optimal/all modules active
```

### LED Patterns
```cpp
// Solid: Continuous illumination
// Flash: 500ms on, 500ms off
// Quick Flash: 200ms on, 200ms off  
// Pulse: Smooth fade in/out over 2 seconds
// Double Flash: 200ms on, 200ms off, 200ms on, 800ms off

PATTERN_SOLID        // Normal operational states
PATTERN_FLASH        // Attention states, warnings
PATTERN_QUICK_FLASH  // Urgent alerts, errors
PATTERN_PULSE        // Standby states, searching
PATTERN_DOUBLE_FLASH // Confirmation states, success
```

## Audio Feedback Specifications

### Sound Types and Uses
```cpp
// Button Feedback
BEEP_BUTTON_PRESS    // 50ms, 2kHz - Button press confirmation
BEEP_BUTTON_INVALID  // 100ms, 1kHz - Invalid button press

// Train Assignment
BEEP_TRAIN_REQUEST   // 3× 200ms, 3kHz - New train assignment
BEEP_TRAIN_CONFIRMED // 2× 100ms, 4kHz - Assignment confirmed
BEEP_TRAIN_REJECTED  // 1× 500ms, 1kHz - Assignment rejected

// System Alerts
BEEP_ERROR           // 3× 300ms, 1.5kHz - System error
BEEP_WARNING         // 2× 200ms, 2.5kHz - System warning
BEEP_STARTUP         // 1× 200ms, 3kHz - System boot complete

// Network Events
BEEP_CONNECTED       // 1× 100ms, 3.5kHz - Network connected
BEEP_DISCONNECTED    // 1× 300ms, 2kHz - Network disconnected
```

### Audio Settings
```cpp
// Volume Control (PWM duty cycle)
VOLUME_OFF    = 0    // Silent operation
VOLUME_LOW    = 25   // Quiet environment
VOLUME_MEDIUM = 50   // Normal operation
VOLUME_HIGH   = 75   // Noisy environment
VOLUME_MAX    = 100  // Emergency/critical alerts
```

## Power Management

### Display Power Management
```cpp
// Display States and Power Consumption
DISPLAY_FULL_BRIGHT  // 40mA - Active use
DISPLAY_DIM          // 20mA - Low activity  
DISPLAY_BACKLIGHT_OFF // 10mA - Content visible, no backlight
DISPLAY_SLEEP        // 2mA - Display controller powered down
DISPLAY_OFF          // 0.5mA - Complete power down

// Auto-timeout Settings
ACTIVE_TIMEOUT       = 300000  // 5 minutes to dim
DIM_TIMEOUT          = 300000  // 5 minutes dim to backlight off  
SLEEP_TIMEOUT        = 600000  // 10 minutes to sleep
```

### User Activity Detection
```cpp
// Activity triggers that reset timeouts
- Button press
- Train assignment received
- System alert/error
- MQTT command received  
- Manual wake (long press any button)
```

### Power Budget
```cpp
Component                | Active  | Standby | Sleep
------------------------|---------|---------|-------
TFT Display             | 40mA    | 20mA    | 2mA
GPIO Expander           | 1mA     | 1mA     | 1mA
Status LEDs (4×)        | 40mA    | 0mA     | 0mA
Buzzer                  | 20mA    | 0mA     | 0mA
------------------------|---------|---------|-------
Total UI Consumption    | 101mA   | 21mA    | 3mA
```

## Implementation Requirements

### Required Libraries
```cpp
#include <TFT_eSPI.h>           // TFT display driver
#include <Wire.h>               // I2C communication
#include <Adafruit_MCP23017.h>  // GPIO expander
#include <ArduinoJson.h>        // JSON message parsing
#include <SPIFFS.h>             // File system for UI assets

// Custom Libraries
#include "ui_manager.h"         // UI state management
#include "screen_manager.h"     // Screen switching
#include "button_handler.h"     // Input processing
#include "led_controller.h"     // Status LED control
#include "buzzer_control.h"     // Audio feedback
```

### Configuration Structure
```cpp
struct UIConfig {
    // Display Configuration
    uint8_t brightness;         // 0-255
    uint16_t timeout_active;    // Seconds
    uint16_t timeout_dim;       // Seconds
    uint16_t timeout_sleep;     // Seconds
    
    // Audio Configuration
    uint8_t volume;             // 0-100
    bool button_sounds;         // Enable button feedback
    bool alert_sounds;          // Enable system alerts
    
    // Visual Configuration
    uint8_t led_brightness;     // 0-255
    bool led_enabled;           // Enable status LEDs
    uint8_t animation_speed;    // 0-255 (0=disabled)
    
    // Interaction Configuration
    uint16_t button_debounce;   // Milliseconds
    uint16_t long_press_time;   // Milliseconds
    uint8_t auto_confirm_time;  // Minutes (0=disabled)
};
```

### Error Handling
```cpp
// Error Recovery Strategies
enum UIError {
    UI_ERROR_DISPLAY_INIT,      // Display initialization failed
    UI_ERROR_GPIO_EXPANDER,     // GPIO expander communication
    UI_ERROR_MEMORY_FULL,       // Insufficient memory for UI
    UI_ERROR_CORRUPT_CONFIG,    // Configuration data corrupted
    UI_ERROR_HARDWARE_FAULT     // Hardware component failure
};

// Recovery Actions
- Display fallback mode (basic text only)
- GPIO expander reinitialize
- Configuration reset to defaults
- Emergency mode operation
- Graceful degradation to core functions
```

## Testing Requirements

### Functional Testing
1. **Display Test**: All pixels functional, color accuracy, brightness control
2. **Button Test**: All buttons responsive, debouncing effective, long press detection
3. **LED Test**: All colors functional, brightness control, pattern generation
4. **Audio Test**: All sound types, volume control, frequency accuracy
5. **Navigation Test**: All screens accessible, transitions smooth, timeout behavior
6. **Train Assignment Test**: Complete workflow from request to confirmation
7. **Error Handling Test**: All error conditions handled gracefully

### Environmental Testing
1. **Temperature Test**: Operation from -20°C to +70°C
2. **Vibration Test**: IEC 61373 railway vibration standards
3. **Humidity Test**: 95% RH non-condensing operation
4. **Electromagnetic Test**: EN 50121 railway EMC compliance
5. **Shock Test**: Drop and impact resistance
6. **UV Test**: Display degradation under sunlight exposure

### User Experience Testing
1. **Visibility Test**: Screen readability under various lighting conditions
2. **Usability Test**: Task completion time, error rates, user satisfaction
3. **Accessibility Test**: Operation with gloves, in dark conditions
4. **Reliability Test**: Extended operation without user intervention
5. **Recovery Test**: System behavior after power cycles, errors

## Maintenance and Troubleshooting

### Diagnostic Features
```cpp
// Built-in Diagnostics
void runUIdiagnostics() {
    // Display Test Pattern
    displayColorBars();
    displayPixelTest();
    
    // Button Response Test
    testAllButtons();
    
    // LED Test Sequence  
    testAllLEDs();
    
    // Audio Test
    testBuzzerTones();
    
    // Communication Test
    testGPIOExpander();
}
```

### Common Issues and Solutions
- **Blank Display**: Check power supply, SPI connections, initialization sequence
- **Unresponsive Buttons**: Verify GPIO expander I2C communication, check debouncing
- **LED Issues**: Confirm GPIO expander pin configuration, check current limiting
- **Audio Problems**: Verify buzzer connections, PWM signal generation
- **Performance Issues**: Check memory usage, optimize rendering, reduce animation complexity

## Future Enhancements

### Planned Improvements
1. **Graphics Library Upgrade**: Higher performance rendering engine
2. **Touch Support**: Optional touch overlay for advanced interaction
3. **Additional Screens**: Maintenance scheduling, route information
4. **Customization**: User-configurable layouts and themes
5. **Multi-language**: Support for Afrikaans, Zulu, other local languages
6. **Data Visualization**: Real-time graphs of speed, location history
7. **Voice Alerts**: Audio announcements for critical events
8. **Remote Configuration**: UI settings configurable via backend system

---

*This specification provides the complete technical foundation for implementing the enhanced UI system, ensuring consistent, reliable, and user-friendly operation in the demanding railway environment.*