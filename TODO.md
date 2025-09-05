# Prompt to run:
since doing the orginal research, i reaslised that procurement might become a problem. thus, it would be better to have a suitable alternative for the base development board. the idea is that both variants should be able to perform all functions and can be interchanged based on product availability. i have identified a suitable alternative, and detailed the differences in the lilygo-comparison-table.md document in the root. read this document carefully and fully integrate the alternative into the existing documentation. take special care to update each and every applicable markdown document in the repo to reflect this alternative. finally, do a comprehensive logic check and cross-reference check of all the documentation to ensure everything makes sense and puts forward a water-tight technical proposal. the aim of the current state of the project is to get a solid framework and gameplan proposal before any actual coding/work is started. be specific on what markdown documents you plan to update/modify/add and how.

continue please, but don't list the R300 savings each time the unit cost is shown. a new reader should not know about the savings, because it is the 'first' and main option presented. rather only mention the R300 INCREASED price when the alternative LILYGO T-SIM7600G-H hardware platform is mentioned. the rest of the edits look great so far.


read both user requirements specifications in the docs/user-specifications folder in detail using python -c "" commands. ignore the satellite-first approach from the LTUSpec.docx (because cellular-first has been adopted), but check if the current state of the project (read and understand every single markdown file in the entire repo) will be a comprehensive solution to the fundamental problem and technical user requirements. make additional suggestions to close any gaps, if required. finally, do a comprehensive logic check and cross-reference check of all the documentation to ensure everything makes sense and puts forward a water-tight technical proposal. the aim of the current state of the project is to get a solid framework and gameplan proposal before any actual coding/work is started. 


# TODO - Locomotive Tracking System

## Development Tasks

### Phase 1: Core Development (Months 1-6)
- [ ] **Hardware Development**
  - [ ] Order LILYGO T-SIM7600G-H development boards
  - [ ] Design enhanced UI hardware (2.8" TFT display, 4-button navigation, status LEDs)
  - [ ] Design PCB layout for production version with UI integration
  - [ ] Create enclosure design for railway environment with front-accessible UI panel
  - [ ] Test power supply integration with 110V railway power (updated for UI: 0.6W idle, 7.7W peak)
  - [ ] Validate GPS performance and antenna placement
  - [ ] Environmental testing (-20°C to +70°C, vibration)

- [ ] **Firmware Development**
  - [ ] Setup PlatformIO development environment
  - [ ] Implement enhanced UI management (TFT display, button navigation, menu system)
  - [ ] Develop train management workflow (assignment confirmation, display)
  - [ ] Implement GPS positioning and tracking logic
  - [ ] Develop cellular communication with SIM7600G
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

- [ ] **Satellite Module Development**
  - [ ] Integrate Swarm satellite modules
  - [ ] Test Iridium module alternative
  - [ ] Implement satellite communication for remote routes
  - [ ] Add global coverage capability

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
- [x] Analyze costs and create business case
- [x] Define architecture and system design
- [x] Create development roadmap and timeline

---

*This TODO list is maintained as a living document throughout the project lifecycle. Tasks are updated based on progress and changing requirements.*