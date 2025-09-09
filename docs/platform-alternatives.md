# Platform Alternatives - Hardware Flexibility Reference

## Overview

This document provides a central reference for hardware platform options in the South African Railway Locomotive GPS Tracking System, ensuring procurement flexibility while maintaining identical functionality across both platforms.

## Platform Comparison

| **Specification** | **T-A7670G R2 Q425** (Primary) | **T-SIM7600G-H** (Alternative) |
|---|---|---|
| **Status** | Primary choice - cost optimized | Backup option - proven platform |
| **Processor** | ESP32-WROVER-E (240MHz dual-core) | ESP32-WROVER-B (240MHz dual-core) |
| **Flash Memory** | 4MB | 4MB |
| **PSRAM** | 8MB | 8MB |
| **Cellular Module** | A7670G (LTE CAT-1) | SIM7600G (LTE CAT-4) |
| **GPS** | L76K GPS module (Q425 version) | Integrated GNSS |
| **Operating Voltage** | 3.4-4.2V direct battery | 5V (via DC-DC converter) |
| **Power Consumption (Idle)** | 70mA @ 3.7V (260mW) | 200mA @ 5V (1000mW) |
| **Power Consumption (Peak)** | 1A @ 3.7V (3.7W) | 1.5A @ 5V (7.5W) |
| **Battery Life** | 48+ hours (10Ah battery) | 30+ hours (10Ah battery) |
| **Unit Price** | R3,375 | R3,675 |
| **100-Unit Fleet Cost** | R337,500 | R367,500 |
| **Network Support** | LTE-FDD/TDD/GSM/GPRS/EDGE | 4G/3G/2G |
| **LTE Frequency Bands** | B1/B2/B3/B4/B5/B7/B8/B12/B13/B18/B19/B20/B25/B26/B28/B66 (LTE-FDD), B38/B39/B40/B41 (LTE-TDD), GSM 850/900/1800/1900MHz | Multiple bands including SA networks |
| **South African Compatibility** | ✅ Full compatibility | ✅ Full compatibility |

## Technical Implementation Differences

### T-A7670G R2 Q425 (Primary Platform)

#### Advantages
- **Power Efficiency**: 65% lower power consumption
- **Cost Optimization**: R300 per unit savings (R30,000 for 100 units)
- **Direct Battery Operation**: Simplified power design
- **Built-in Battery Holder**: More integrated design
- **Solar Charging Interface**: Built-in solar capability

#### Implementation Details
- **Cellular AT Commands**: A7670G command set
- **GPS Interface**: Separate L76K module via UART
- **Power Supply**: Direct 3.4-4.2V battery operation
- **Libraries**: A7670G cellular drivers + L76K GPS drivers
- **GPS Reliability**: L76K requires robust driver with retry logic
- **Network Speeds**: CAT-1 (10Mbps DL/5Mbps UL) - adequate for telemetry

### T-SIM7600G-H (Alternative Platform)

#### Advantages  
- **Proven Track Record**: Extensive deployment history
- **Integrated GPS**: Single module for cellular + GPS
- **Higher Bandwidth**: CAT-4 vs CAT-1 (not required for telemetry)
- **Documentation**: More extensive examples and tutorials

#### Implementation Details
- **Cellular AT Commands**: SIM7600G command set  
- **GPS Interface**: Integrated GPS via cellular module
- **Power Supply**: Requires DC-DC converter for 5V operation
- **Libraries**: SIM7600G cellular drivers (GPS included)
- **GPS Reliability**: Integrated GPS with proven track record
- **Network Speeds**: CAT-4 (150Mbps DL/50Mbps UL) - higher than required

## Procurement Strategy

### Primary Approach
1. **Source T-A7670G R2 Q425** as primary platform for cost and power benefits
2. **Order pilot quantities** (10-20 units) for initial testing and validation
3. **Validate functionality** across all required features before bulk ordering

### Backup Strategy  
1. **Maintain T-SIM7600G-H** as secondary source for supply chain resilience
2. **Pre-validate alternative** with parallel development and testing
3. **Document migration process** for rapid switching if procurement issues arise

### Risk Mitigation
- **Dual Source Validation**: Both platforms tested and proven
- **Identical Functionality**: No feature compromises regardless of platform choice
- **Rapid Switching**: Minor firmware changes enable quick platform change
- **Cost Protection**: Budget includes alternative platform costs
- **GPS Reliability Strategy**: L76K driver with retry logic, external GPS option, extensive field validation
- **Network Compatibility**: Both platforms fully compatible with SA cellular networks (MTN/Vodacom)

## Development Considerations

### Firmware Modifications for Platform Switching

#### T-A7670G → T-SIM7600G-H Changes
- **AT Command Set**: Update cellular communication commands
- **GPS Handling**: Switch from L76K module to integrated GPS
- **Power Management**: Add 5V power supply handling
- **Library Dependencies**: Replace A7670G drivers with SIM7600G equivalents
- **Estimated Effort**: 2-3 days development + testing

#### T-SIM7600G-H → T-A7670G Changes  
- **AT Command Set**: Update to A7670G command structure
- **GPS Handling**: Implement separate L76K module communication
- **Power Management**: Update for direct battery operation
- **Library Dependencies**: Replace SIM7600G drivers with A7670G equivalents
- **Estimated Effort**: 2-3 days development + testing

### Common Components (Unchanged)
- **Enhanced UI System**: Identical 2.8" TFT display and navigation
- **GPIO Expander**: Same MCP23017 I2C 16-port expander
- **Train Management**: Identical MQTT commands and workflows
- **Expansion Modules**: Same LoRa and satellite module interfaces
- **OTA Updates**: Identical update mechanism and security
- **Debug Mode**: Same WiFi Access Point functionality

## Fleet Deployment Scenarios

### Scenario 1: Single Platform (Recommended)
- Deploy 100 units using T-A7670G R2 Q425
- **Cost**: R337,500 hardware + R180,000 development = R517,500
- **Benefits**: Simplified inventory, optimized power consumption, lower costs

### Scenario 2: Mixed Fleet (Contingency)
- Deploy 70 units T-A7670G + 30 units T-SIM7600G-H due to supply constraints  
- **Cost**: R337,500 + R110,250 hardware + R185,000 development = R632,750
- **Benefits**: Supply chain resilience, identical functionality

### Scenario 3: Alternative Platform (Backup)
- Deploy 100 units using T-SIM7600G-H if T-A7670G unavailable
- **Cost**: R367,500 hardware + R180,000 development = R547,500  
- **Benefits**: Proven platform, extensive documentation

## Quality Assurance

### Testing Requirements (Both Platforms)
- **Functional Testing**: GPS accuracy, cellular connectivity, train management workflows
- **Power Testing**: Battery life validation, power consumption measurement
- **Environmental Testing**: Temperature range, vibration resistance, EMC compliance
- **Integration Testing**: Backend connectivity, MQTT communication, OTA updates
- **Field Testing**: Real locomotive deployment validation

### Acceptance Criteria (Platform Independent)
- ✅ GPS accuracy within 3 meters
- ✅ Cellular connectivity >95% uptime
- ✅ 48-hour battery backup minimum  
- ✅ Train assignment workflow <30 second response
- ✅ OTA update success rate >99%
- ✅ Enhanced UI response time <200ms
- ✅ Debug mode data extraction functionality

## Support and Documentation

### T-A7670G R2 Q425 Resources
- **Official Documentation**: LILYGO technical specifications
- **Community Support**: Growing developer community
- **Library Support**: A7670G Arduino libraries available
- **Application Notes**: Power optimization guides
- **GPS Considerations**: L76K may have longer cold start times, requires separate UART implementation

### T-SIM7600G-H Resources  
- **Extensive Documentation**: Comprehensive examples and tutorials
- **Mature Community**: Large developer ecosystem
- **Proven Libraries**: Well-tested SIM7600G drivers
- **Application Support**: Wide range of implementation examples

## Conclusion

The dual-platform approach provides optimal flexibility:
- **T-A7670G R2 Q425** offers superior power efficiency and cost optimization
- **T-SIM7600G-H** provides proven reliability and extensive documentation
- **Both platforms** support identical functionality with minimal firmware differences
- **Procurement flexibility** ensures project success regardless of supply chain challenges

This strategy delivers a robust, cost-effective solution while maintaining the ability to adapt to changing procurement conditions without compromising functionality or timeline.

## Future Technology Integration: Starlink

**Modular Architecture Benefit**: The system's modular design enables seamless integration with emerging satellite technologies like Starlink once available in South Africa.

**Key Advantages:**
- **Cost Revolution**: Starlink may provide satellite connectivity at a fraction of current costs (vs R18,550/month for ICASA-approved Iridium)
- **Performance Leap**: High-speed, low-latency connectivity enabling advanced railway applications
- **Module Compatibility**: Both T-A7670G R2 Q425 and T-SIM7600G-H platforms support expansion modules
- **No Hardware Replacement**: Existing installations can be upgraded with Starlink modules when available
- **Strategic Positioning**: Ready for immediate adoption upon South African service launch

This demonstrates the forward-looking value of the modular platform approach, ensuring long-term technology relevance and investment protection.

## Regulatory Compliance Considerations

### Satellite Communication Modules

Both hardware platforms support identical expansion module interfaces, including satellite communication modules. Current regulatory status in South Africa:

**ICASA-Approved Services**:
- **Iridium 9603N**: Fully approved, immediate deployment ready
- **Cost**: R18,550/month per unit at 1-minute intervals

**Pending ICASA Approval**:
- **Astrocast Astronode S**: Potentially cost-effective (R5,123/month) pending approval
- **Swarm M138**: Alternative option (R3,309/month) pending approval

**Implementation Strategy**:
- Deploy with ICASA-approved Iridium modules for compliance
- Monitor regulatory developments for future cost optimization opportunities
- Modular architecture enables quick transition to approved alternatives

This ensures both platform options maintain regulatory compliance while preserving flexibility for future optimization.

---

*For specific implementation details, refer to hardware documentation in `hardware/README.md` and firmware specifications in `firmware/README.md`.*