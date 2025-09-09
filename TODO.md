# Prompts to run:

* read both user requirements specifications in the docs/user-specifications folder in detail using python -c "" commands. ignore the satellite-first approach from the LTUSpec.docx (because cellular-first has been adopted), but check if the current state of the project (read and understand every single markdown file in the entire repo) will be a comprehensive solution to the fundamental problem and technical user requirements. make additional suggestions to close any gaps, if required. finally, do a comprehensive logic check and cross-reference check of all the documentation to ensure everything makes sense and puts forward a water-tight technical proposal. the aim of the current state of the project is to get a solid framework and gameplan proposal before any actual coding/work is started. be specific on what markdown documents you plan to update/modify/add and how.


# TODO - Locomotive Tracking System

## Development Tasks

### Phase 1: Core Development (Months 1-6)
- [ ] **Hardware Development**
  - [ ] Order LILYGO T-A7670G R2 Q425 development boards
  - [ ] Design enhanced UI hardware (2.8" TFT display, 4-button navigation, status LEDs)
  - [ ] Design PCB layout for production version with UI integration
  - [ ] Create enclosure design for railway environment with front-accessible UI panel
  - [ ] Test power supply integration with 110V railway power (updated for UI: 0.26W idle, 3.7W peak)
  - [ ] Validate GPS performance and antenna placement
  - [ ] Environmental testing (-20°C to +70°C, vibration)

- [ ] **Firmware Development**
  - [ ] Setup PlatformIO development environment
  - [ ] Implement enhanced UI management (TFT display, button navigation, menu system)
  - [ ] Develop train management workflow (assignment confirmation, display)
  - [ ] Implement GPS positioning and tracking logic
  - [ ] Develop cellular communication with LILYGO T-A7670G R2 Q425
  - [ ] Implement MQTT client with TLS security for train assignment commands
  - [ ] Create power management and sleep modes (updated for enhanced UI)
  - [ ] Add data logging to SD card
  - [ ] Implement configuration management
  - [ ] Add OTA update capability

- [ ] **Backend Development**
  - [ ] Setup .NET Core Web API project
  - [ ] Design database schema with spatial extensions and train management tables
  - [ ] Develop train management integration services (ITP, TMS, VDU connectors)
  - [ ] Create integration gateway for Transnet system APIs
  - [ ] Implement MQTT broker integration with train assignment commands
  - [ ] Create device management APIs and train management APIs
  - [ ] Develop real-time SignalR hubs for train assignments
  - [ ] Add authentication and authorization
  - [ ] Implement background data processing and train synchronization services
  - [ ] Create reporting and analytics features

- [ ] **Frontend Development**
  - [ ] Setup React.js dashboard project
  - [ ] Integrate mapping library (Leaflet/OpenLayers)
  - [ ] Create real-time locomotive position display with train identification
  - [ ] Build device management interface with train assignment controls
  - [ ] Add fleet status monitoring with train number displays
  - [ ] Develop train management interface for TCO workflow
  - [ ] Implement alert and notification system
  - [ ] Create reporting dashboards with train analytics

### Phase 2: Testing & Validation (Months 6-12)
- [ ] **Laboratory Testing**
  - [ ] GPS accuracy validation
  - [ ] Cellular connectivity testing (4G/3G/2G)
  - [ ] Power consumption analysis
  - [ ] Temperature and vibration testing
  - [ ] EMC compliance testing

- [ ] **Field Testing**
  - [ ] Deploy 10 pilot units on test locomotives
  - [ ] Validate coverage on different route types
  - [ ] Test failover scenarios (cellular network issues)
  - [ ] Measure data usage and optimize compression
  - [ ] Validate battery backup performance

- [ ] **Integration Testing**
  - [ ] End-to-end data flow validation including train management
  - [ ] Transnet system integration testing (ITP, TMS, VDU connectivity)
  - [ ] Train assignment workflow testing (TCO to device confirmation)
  - [ ] Dashboard real-time update testing with train identification
  - [ ] Multi-device performance testing
  - [ ] Security penetration testing
  - [ ] Load testing with 100+ simulated devices

### Phase 3: Production Deployment (Months 12-18)
- [ ] **Manufacturing**
  - [ ] Source components for 100 production units
  - [ ] Setup assembly process and quality control
  - [ ] Create installation documentation
  - [ ] Train installation technicians

- [ ] **System Deployment**
  - [ ] Setup production infrastructure (servers, databases)
  - [ ] Configure private APN with cellular providers
  - [ ] Deploy MQTT broker and security certificates
  - [ ] Create device provisioning system
  - [ ] Implement monitoring and alerting

- [ ] **Fleet Rollout**
  - [ ] Install units on high-priority locomotives
  - [ ] Configure route profiles for different operations
  - [ ] Train operations staff on dashboard usage
  - [ ] Establish maintenance procedures
  - [ ] Complete system acceptance testing

## Future Enhancements (Phase 4+)

### Modular Expansion
- [ ] **LoRa Module Development**
  - [ ] Design LoRa communication module
  - [ ] Install gateway infrastructure at depots
  - [ ] Implement cost-optimized depot communication
  - [ ] Add automatic cellular/LoRa failover

- [ ] **Satellite Module Development (REGULATORY COMPLIANT)**
  - [ ] **CRITICAL**: Use only ICASA-approved satellite services (Iridium 9603N)
  - [ ] Limit satellite to emergency communications only (R18,550/month per unit - Iridium baseline)
  - [ ] Integrate 1-2 Iridium satellite modules for testing
  - [ ] Position satellite as last resort, regulatory-compliant emergency communication
  - [ ] Monitor ICASA approval status for Astrocast and Swarm for future cost optimization

### Industrial-Grade Hardware
- [ ] **Custom Carrier Board Design**
  - [ ] Design custom PCB with industrial-grade components (Quectel BG95-M3, ESP32-S3).
  - [ ] Implement EN50155 compliant power supply and M12 connectors.
  - [ ] Add conformal coating and enhanced protection circuits.
- [ ] **Manufacturing and Sourcing**
  - [ ] Source industrial-grade components for mass production.
  - [ ] Establish partnership with a PCB assembly house.
- [ ] **Deployment**
  - [ ] Plan phased rollout of industrial-grade units to replace prototypes.

### Advanced Features
- [ ] **AI and Analytics**
  - [ ] Predictive maintenance algorithms
  - [ ] Route optimization recommendations
  - [ ] Anomaly detection for security
  - [ ] Driver behavior analysis

- [ ] **Integration Enhancements**
  - [ ] ERP system integration
  - [ ] Maintenance management integration
  - [ ] Fuel monitoring system integration
  - [ ] Wagon tracking integration

## Critical Tasks (Immediate - Regulatory Compliance)
1. [x] **COMPLETED**: Implemented regulatory-compliant satellite strategy with Iridium 9603N baseline
2. [x] **COMPLETED**: Updated budget projections with regulatory-compliant costs (5-year: R6.4M)
3. [x] **COMPLETED**: Revised satellite usage to emergency communications only (R18,550/month per unit)
4. [ ] **URGENT**: Monitor ICASA approval developments for Astrocast and Swarm services
5. [x] **COMPLETED**: Updated all project documentation with regulatory-compliant satellite costs
6. [x] **COMPLETED**: Updated project cost rankings with regulatory baseline costs
7. [x] **COMPLETED**: Reduced satellite module deployment to emergency use only (1-2 units maximum)

## Priority Tasks (Next 30 Days)
1. [ ] Order development hardware (LILYGO boards, antennas, power supplies)
2. [ ] Setup development environments (PlatformIO, .NET Core, React)
3. [ ] Create initial firmware skeleton with GPS and cellular
4. [ ] Setup database server with spatial extensions
5. [ ] Configure test MQTT broker
6. [ ] Create basic dashboard with map display
7. [ ] Establish cellular provider contacts for APN setup

## Known Issues & Risks
- [ ] **Hardware Risks**
  - [ ] LILYGO board availability and lead times
  - [ ] Component cost fluctuations
  - [ ] Environmental certification requirements

- [ ] **Technical Risks**
  - [ ] Cellular coverage gaps on remote routes
  - [x] **COMPLETED**: Satellite communication costs addressed with regulatory-compliant Iridium baseline (R18,550/month per unit)
  - [x] **COMPLETED**: Positioned satellite as emergency-only solution due to high costs
  - [ ] Data cost management and optimization
  - [ ] Battery life in extreme temperatures

- [ ] **Business Risks**
  - [ ] Private APN setup complexity
  - [ ] Regulatory approval timelines
  - [ ] Staff training and adoption

## Completed Tasks
- [x] Research and select optimal implementation approach
- [x] Create comprehensive project structure
- [x] Document technical specifications
- [x] **UPDATED**: Analyze costs and create business case (satellite costs corrected across all documentation)
- [x] Define architecture and system design
- [x] Create development roadmap and timeline
- [x] **COMPLETED**: Comprehensive satellite cost analysis with regulatory compliance focus
- [x] **COMPLETED**: Updated all project documentation with ICASA-approved Iridium costs (R18,550/month per unit)
- [x] **COMPLETED**: Revised strategic positioning with regulatory-compliant satellite baseline
- [x] **COMPLETED**: Added potential cost reduction scenarios if Astrocast/Swarm receive ICASA approval

---

*This TODO list is maintained as a living document throughout the project lifecycle. Tasks are updated based on progress and changing requirements.*