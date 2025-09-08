# Contributing to Locomotive Tracking System

Thank you for your interest in contributing to the South African Railway Locomotive GPS Tracking System. This document provides guidelines for contributing to the project.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Process](#development-process)
4. [Code Standards](#code-standards)
5. [Testing Requirements](#testing-requirements)
6. [Documentation](#documentation)
7. [Hardware Contributions](#hardware-contributions)
8. [Pull Request Process](#pull-request-process)

## Code of Conduct

This project is developed for critical railway infrastructure. All contributors must:

- Follow professional development practices
- Prioritize safety and reliability in all implementations
- Respect the operational requirements of railway systems
- Maintain confidentiality of sensitive operational information
- Test thoroughly before submitting changes

## Getting Started

### Prerequisites

1. **Hardware Development**
   - KiCad 6.0+ for PCB design
   - LILYGO T-A7670G R2 Q425 development boards for testing

2. **Firmware Development**
   - PlatformIO for ESP32 development
   - Git for version control
   - Serial terminal for debugging

3. **Backend Development**
   - .NET 8 SDK
   - SQL Server or LocalDB
   - Docker for local services

4. **Frontend Development**
   - Node.js 18+
   - React.js knowledge
   - Mapping library experience (Leaflet/OpenLayers)

### Development Environment Setup

```bash
# Clone the repository
git clone https://github.com/your-org/locomotive-tracking-unit.git
cd locomotive-tracking-unit

# Setup firmware development
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

## Development Process

### Branching Strategy

- **main** - Production-ready code
- **develop** - Integration branch for features
- **feature/\*** - Individual feature branches
- **hotfix/\*** - Critical bug fixes
- **release/\*** - Release preparation branches

### Workflow

1. Create feature branch from `develop`
2. Implement changes with appropriate tests
3. Ensure all tests pass
4. Update documentation as needed
5. Submit pull request to `develop` branch
6. Address review feedback
7. Merge after approval

## Code Standards

### Firmware (C++)

```cpp
// File header required
/*
 * File: gps_manager.cpp
 * Purpose: GPS positioning and tracking functionality
 * Author: Your Name
 * Date: YYYY-MM-DD
 * Safety Critical: Yes
 */

// Naming conventions
class GPSManager {           // PascalCase for classes
private:
    float latitude_;         // snake_case for private members
    bool is_valid_;         // Descriptive boolean names
    
public:
    bool IsPositionValid();  // PascalCase for public methods
    void UpdatePosition();   // Clear, action-oriented names
};

// Constants in ALL_CAPS
const int GPS_TIMEOUT_MS = 5000;
const float MIN_VALID_SPEED = 0.1;

// Error handling required
if (!gps.begin()) {
    LOG_ERROR("Failed to initialize GPS module");
    return false;
}
```

### Backend (C#)

```csharp
// Follow .NET conventions
public class DeviceService : IDeviceService
{
    private readonly IDeviceRepository _deviceRepository;
    private readonly ILogger<DeviceService> _logger;
    
    public async Task<Device> GetDeviceAsync(int deviceId)
    {
        try
        {
            return await _deviceRepository.GetByIdAsync(deviceId);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to retrieve device {DeviceId}", deviceId);
            throw;
        }
    }
}

// Use dependency injection
services.AddScoped<IDeviceService, DeviceService>();
```

### Frontend (JavaScript/React)

```javascript
// Use modern React patterns
const DeviceList = ({ devices, onDeviceSelect }) => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadDevices();
  }, []);

  const loadDevices = async () => {
    try {
      setLoading(true);
      const data = await deviceApi.getAll();
      setDevices(data);
    } catch (err) {
      setError('Failed to load devices');
      console.error('Device loading error:', err);
    } finally {
      setLoading(false);
    }
  };

  // Component implementation...
};
```

## Testing Requirements

### Firmware Testing

- **Unit Tests**: Test individual functions and classes
- **Integration Tests**: Test hardware component interactions
- **Hardware-in-Loop**: Test with actual railway conditions

```cpp
// Example unit test
#include <gtest/gtest.h>

TEST(GPSManagerTest, ValidateCoordinates) {
    GPSManager gps;
    
    // Test valid coordinates
    EXPECT_TRUE(gps.IsValidCoordinate(-26.195246, 28.034088));
    
    // Test invalid coordinates
    EXPECT_FALSE(gps.IsValidCoordinate(999.0, 999.0));
}
```

### Backend Testing

- **Unit Tests**: xUnit for business logic
- **Integration Tests**: Test with real database
- **API Tests**: Test endpoints with WebApplicationFactory

```csharp
[Fact]
public async Task GetDevice_ValidId_ReturnsDevice()
{
    // Arrange
    var deviceId = 1;
    var expectedDevice = new Device { Id = deviceId, IMEI = "123456789" };
    _mockRepository.Setup(r => r.GetByIdAsync(deviceId))
                  .ReturnsAsync(expectedDevice);

    // Act
    var result = await _deviceService.GetDeviceAsync(deviceId);

    // Assert
    Assert.NotNull(result);
    Assert.Equal(expectedDevice.IMEI, result.IMEI);
}
```

### Frontend Testing

- **Unit Tests**: Jest for components and utilities
- **Integration Tests**: React Testing Library
- **E2E Tests**: Cypress for critical user flows

## Documentation

### Code Documentation

- **Comment complex algorithms and business logic**
- **Document safety-critical functions thoroughly**
- **Include examples for public APIs**
- **Update README files when adding new features**

### API Documentation

- **Use XML documentation comments for .NET**
- **Include request/response examples**
- **Document error conditions and status codes**
- **Keep OpenAPI/Swagger documentation updated**

### Hardware Documentation

- **Include schematic diagrams for circuit changes**
- **Document component selection rationale**
- **Provide assembly instructions**
- **Include testing procedures**

## Hardware Contributions

### PCB Design Guidelines

1. **Follow railway EMC standards** (EN 50121)
2. **Use appropriate component ratings** for railway environment
3. **Include adequate protection circuits** (TVS diodes, fuses)
4. **Design for serviceability** (test points, indicators)
5. **Consider manufacturing constraints** (component availability, assembly)

### Mechanical Design

1. **Design for IP67 protection rating**
2. **Account for vibration resistance** (IEC 61373)
3. **Consider thermal management** (-20°C to +70°C operation)
4. **Design for maintenance access**
5. **Use railway-approved materials**

## Pull Request Process

### Before Submitting

- [ ] All tests pass locally
- [ ] Code follows project standards
- [ ] Documentation updated
- [ ] No security issues introduced
- [ ] Performance impact considered

### Pull Request Template

```markdown
## Description
Brief description of changes and motivation.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Hardware change
- [ ] Documentation update
- [ ] Performance improvement

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Hardware tested (if applicable)
- [ ] Manual testing completed

## Safety Impact
- [ ] No safety impact
- [ ] Safety analysis completed
- [ ] Safety review required

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] Tests added for new functionality
```

### Review Process

1. **Automated checks** must pass (CI/CD pipeline)
2. **Code review** by at least one team member
3. **Safety review** for safety-critical changes
4. **Hardware review** for hardware modifications
5. **Integration testing** in development environment

### Merge Requirements

- All automated checks pass
- At least one approval from code owner
- No outstanding review comments
- Documentation updated appropriately

## Questions and Support

For questions about contributing:

1. **Technical Questions**: Create an issue with the `question` label
2. **Hardware Questions**: Contact the hardware team lead
3. **Safety Questions**: Consult the safety engineer
4. **Process Questions**: Review this document or ask in team meetings

## Recognition

Contributors will be recognized in:
- Project documentation
- Release notes
- Team meetings
- Annual project reviews

Thank you for contributing to railway safety and operational efficiency!

---

*This contributing guide ensures quality, safety, and maintainability of the locomotive tracking system.*