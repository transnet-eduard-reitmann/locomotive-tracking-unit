# South African Railway Locomotive GPS Tracking System

A comprehensive GPS tracking system for locomotive fleet management across the South African rail network, providing real-time location data, route optimization, and comprehensive fleet visibility.

## Project Overview

This project offers **three distinct implementation variants** to suit different operational requirements, budget constraints, and coverage needs for railway locomotive tracking in South Africa.

### 🚂 **Variant 1: [Cellular-Only](docs/cellular_only_option.md)**
*Simple, proven, real-time tracking*
- **Hardware Cost:** R2,870/unit
- **Monthly Cost:** R50/unit  
- **Timeline:** 15 months
- **Coverage:** 99%+ real-time
- **Best for:** Immediate deployment, real-time operations, proven reliability

### 📡 **Variant 2: [LoRa-Only](docs/lora_only_option.md)**  
*Cost-optimized store-and-forward*
- **Hardware Cost:** R1,550/unit
- **Monthly Cost:** R0 (zero operational costs)
- **Timeline:** 24 months  
- **Coverage:** 30-90% delayed data
- **Best for:** Budget-conscious deployments, comprehensive historical tracking

### 🌐 **Variant 3: [Cellular + LoRa Hybrid](docs/cellular_and_lora_option.md)**
*Maximum reliability and redundancy*
- **Hardware Cost:** R3,220/unit
- **Monthly Cost:** R50/unit
- **Timeline:** 18 months
- **Coverage:** 99%+ with backup redundancy
- **Best for:** Mission-critical operations, maximum uptime requirements

---

## Quick Comparison

| Aspect | Cellular-Only | LoRa-Only | Hybrid |
|--------|---------------|-----------|--------|
| **Real-time Capability** | ✅ Excellent | ❌ Limited | ✅ Excellent |
| **Coverage** | 99%+ | 30-90% | 99%+ |
| **Initial Cost (100 units)** | R287K | R155K | R322K |
| **5-Year Total Cost** | R937K | R551K | R1,343K |
| **Deployment Complexity** | Low | High | Medium |
| **Infrastructure Required** | None | Gateway Network | Gateway Network |
| **Emergency Response** | Immediate | Delayed | Immediate |
| **Operational Independence** | Carrier Dependent | Self-Sufficient | Hybrid |

---

## Technical Foundation

### Core Hardware Platform
All variants are built on proven, industrial-grade hardware:
- **ESP32-based processing** for reliable embedded computing
- **Multi-GNSS GPS** (GPS, GLONASS, BeiDou, Galileo) for accuracy
- **Railway-grade enclosures** (IP67, vibration resistant, -20°C to +70°C)
- **Locomotive power integration** with 48-hour battery backup
- **Secure data logging** with local storage capabilities

### Unified Software Architecture
- **Modular firmware** supporting all communication variants
- **.NET Core backend** with spatial database integration
- **Real-time dashboard** with mapping and fleet management
- **MQTT protocol** for secure, efficient data transmission
- **OTA update capability** for remote maintenance

### South African Compliance
- **ICASA certification** for all radio frequency components
- **Railway Safety Regulator** compliance for operational deployment
- **POPI Act compliance** for location data privacy
- **Multi-carrier support** (MTN, Vodacom, Cell C, Telkom)

---

## Implementation Strategy

### Choose Your Approach

#### **Start Simple → Scale Smart**
1. **Begin with Cellular-Only** for immediate operational capability
2. **Add LoRa infrastructure** in phases for cost optimization
3. **Migrate selective units** to hybrid approach for critical routes

#### **Infrastructure-First → Cost-Optimize**
1. **Deploy LoRa gateway network** for long-term cost benefits
2. **Roll out LoRa-only units** for majority of fleet
3. **Add cellular capability** only where real-time response is critical

#### **Maximum Reliability → Mission-Critical**
1. **Full hybrid deployment** for maximum uptime and redundancy
2. **Complete coverage** with multiple communication paths
3. **Enterprise-grade** reliability for high-value operations

---

## Project Structure

```
locomotive-tracker/
├── docs/                              # Comprehensive documentation
│   ├── cellular_only_option.md       # Variant 1 specifications
│   ├── lora_only_option.md           # Variant 2 specifications
│   ├── cellular_and_lora_option.md   # Variant 3 specifications
│   └── implementation_comparison.md   # Detailed variant analysis
├── firmware/                          # Embedded software
│   ├── common/                        # Shared libraries
│   ├── cellular_only/                 # Variant 1 firmware
│   ├── lora_only/                     # Variant 2 firmware
│   ├── hybrid/                        # Variant 3 firmware
│   └── gateway/                       # LoRa gateway firmware
├── backend/                           # Server-side applications
│   ├── api/                          # .NET Core Web API
│   ├── database/                     # SQL Server integration
│   └── mqtt_broker/                  # Message handling
├── frontend/                          # User interfaces
│   ├── dashboard/                    # Web-based fleet management
│   └── mobile/                       # Mobile applications
├── hardware/                          # Hardware designs
│   ├── cellular_only/                # Variant 1 PCB designs
│   ├── lora_only/                    # Variant 2 PCB designs
│   ├── hybrid/                       # Variant 3 PCB designs
│   └── common/                       # Shared components
└── tools/                            # Development utilities
    ├── data_analysis/                # Analytics scripts
    ├── simulation/                   # Coverage modeling
    └── utilities/                    # Helper tools
```

---

## Getting Started

### 1. **Evaluate Requirements**
- Review operational needs for real-time vs. historical tracking
- Assess budget constraints for initial and ongoing costs  
- Determine coverage requirements across route network
- Consider existing IT infrastructure and capabilities

### 2. **Choose Variant**
- **Need immediate real-time tracking?** → Cellular-Only
- **Want zero operational costs?** → LoRa-Only  
- **Require maximum reliability?** → Hybrid

### 3. **Development Setup**
```bash
# Clone repository
git clone https://github.com/yourusername/locomotive-tracker.git
cd locomotive-tracker

# Setup PlatformIO for firmware development
pip install platformio
pio init

# Setup .NET development environment
dotnet restore backend/api/
dotnet build backend/api/
```

### 4. **Hardware Prototyping**
- Order development kits for chosen variant
- Setup test environment with GPS and communication modules
- Validate functionality in controlled environment
- Plan field testing procedures

---

## Investment Analysis

### Total Cost of Ownership (100 Units, 5 Years)

| Variant | Hardware | Development | Infrastructure | Operational | **Total** |
|---------|----------|-------------|----------------|-------------|-----------|
| **Cellular-Only** | R287K | R350K | R0 | R300K | **R937K** |
| **LoRa-Only** | R155K | R366K | R0 | R30K | **R551K** |
| **Hybrid** | R322K | R691K | R0 | R330K | **R1,343K** |

### ROI Considerations
- **Operational efficiency** through improved fleet utilization
- **Fuel savings** from route optimization (estimated 5-10%)
- **Maintenance cost reduction** through predictive analytics
- **Regulatory compliance** and safety improvements
- **Asset security** and theft prevention

---

## Risk Assessment

### Technical Risks
| Risk | Cellular | LoRa | Hybrid | Mitigation |
|------|----------|------|--------|------------|
| **Coverage gaps** | Low | High | Low | Multi-path communication |
| **Operational costs** | High | None | High | Usage optimization |
| **Infrastructure complexity** | Low | High | Medium | Professional deployment |
| **Technology obsolescence** | Medium | Low | Medium | Modular design |

### Business Risks
- **Regulatory changes** - Mitigated through compliant design and ICASA certification
- **Vendor dependencies** - Reduced through multi-supplier approach
- **Scalability challenges** - Addressed through modular architecture
- **Integration complexity** - Managed through phased deployment

---

## Contributing

This project welcomes contributions in:
- **Hardware design improvements** and cost optimizations
- **Firmware development** for enhanced functionality
- **Backend services** and API enhancements  
- **Documentation** and implementation guides
- **Testing procedures** and field validation

### Development Guidelines
1. Follow modular design principles for cross-variant compatibility
2. Maintain backwards compatibility across firmware versions
3. Document all hardware modifications and electrical changes
4. Test thoroughly in railway environments before production
5. Ensure compliance with all South African regulatory requirements

---

## Support & Documentation

### Documentation
- 📋 **[Cellular-Only Implementation](docs/cellular_only_option.md)** - Complete Variant 1 guide
- 📡 **[LoRa-Only Implementation](docs/lora_only_option.md)** - Complete Variant 2 guide  
- 🌐 **[Hybrid Implementation](docs/cellular_and_lora_option.md)** - Complete Variant 3 guide
- 📊 **[Implementation Comparison](docs/implementation_comparison.md)** - Detailed variant analysis

### Project Resources
- **GitHub Pages:** [https://yourusername.github.io/locomotive-tracker](https://yourusername.github.io/locomotive-tracker)
- **Issues & Support:** [GitHub Issues](https://github.com/yourusername/locomotive-tracker/issues)
- **Discussions:** [GitHub Discussions](https://github.com/yourusername/locomotive-tracker/discussions)

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- **South African Railway Industry** for operational insights and requirements
- **LILYGO** for reliable ESP32 hardware platforms
- **LoRaWAN Community** for protocol development and support
- **South African Cellular Operators** for IoT connectivity solutions

---

*Developed for the South African railway industry with focus on practical, cost-effective, and reliable locomotive tracking solutions.*