# Integrated Locomotive Tracking Unit - Specification (Revised)

## 1. Scope

### 1.1. Identification
This document specifies the requirements for the Integrated Locomotive Tracking Unit (LTU).

### 1.2. System Overview
The LTU is a critical component of a larger system designed to provide real-time tracking, identification, and data communication for locomotives within the Transnet fleet. The LTU will be installed on each locomotive, providing a reliable stream of position data and serving as a communication hub. This data is essential for solving the core business problem of associating a specific locomotive and its vehicle list with a correct train number in real-time.

The system is based on a **Cellular-First, Modular Expansion** architecture.

---

## 2. Functional Requirements

### 2.1. Positioning
- **2.1.1.** The LTU shall determine its position using a Global Navigation Satellite System (GNSS).
- **2.1.2.** The LTU shall measure position continuously while powered.
- **2.1.3.** Each recorded position shall be time-stamped. Synchronization with a reliable network time source is required.

### 2.2. Communication
- **2.2.1. Communication Strategy:** The LTU shall employ a "Cellular-First" communication strategy, prioritizing the most cost-effective and efficient communication channel available.
- **2.2.2. Primary Communication (Cellular):** The LTU shall use a 4G/3G/2G cellular connection as the primary means of transmitting all data, including real-time position data. This is the default communication method when a cellular network is available.
- **2.2.3. Secondary Communication (LoRa - Optional):** For cost-optimization in depot and yard environments, an optional LoRa module may be fitted. When in range of a LoRa gateway, the LTU shall use LoRa for data transmission.
- **2.2.4. Tertiary Communication (Satellite - Optional):** For operation in remote areas with no terrestrial network coverage, an optional satellite module may be fitted. The LTU shall use the satellite connection when neither Cellular nor LoRa networks are available.
- **2_2.5. Data Buffering:** In the event of a total communication failure, the LTU shall store all data in local non-volatile memory. It shall automatically transmit the buffered data in chronological order once a communication link is re-established.
- **2.2.6. OTA Updates:** The LTU shall support over-the-air (OTA) software and firmware updates, primarily delivered via the cellular network.

### 2.3. Onboard User Interface
- **2.3.1.** The LTU shall have a simple onboard display and input interface for the train driver.
- **2.3.2. Display:** The display shall clearly show the configured Locomotive Number and the currently assigned Train Number.
- **2.3.3. Input:** The input interface shall consist of at least three physical buttons: "Up", "Down", and "OK" to allow the user to scroll through information and confirm actions.
- **2.3.4. Train Number Workflow:** The LTU shall support a workflow for the driver to confirm train number assignments and deactivations received from the central server via the onboard interface.

### 2.4. Data and Logging
- **2.4.1.** The LTU shall log and send position data at a configurable interval (e.g., 10s, 30s, 60s), with the interval being adaptable based on locomotive speed and operational status.
- **2.4.2.** Each outbound message shall include the LTU's unique device ID and the configured locomotive number.
- **2.4.3.** The LTU shall maintain a rolling log of the last 7 days of position reports on local storage.
- **2_4.4.** Logs shall be downloadable via a local interface (e.g., USB) for forensic or compliance purposes.

---

## 3. Performance Requirements
- **3.1. Positional Accuracy:** Shall be within 5 meters Circular Error Probability (CEP).
- **3.2. System Availability:** Shall achieve >99% availability during any 30-day period.
- **3.3. Data Latency:** Position data must be transmitted and received by the central server within 60 seconds under normal cellular operating conditions.
- **3.4. Local Storage:** Shall have at least 32GB of non-volatile memory.
- **3.5. Server Capacity:** The central server and communication protocol must support at least 5,000 concurrent LTUs without degradation in response time.
- **3.6. User Interface Response:** The onboard screen and buttons shall respond to input within 1 second.
- **3.7. Time to First Fix:** The LTU shall acquire a valid GNSS position fix within 2 minutes of a cold start.

---

## 4. Operational and Maintenance Requirements

### 4.1. Environmental Hardening
- **4.1.1.** The LTU shall be environmentally hardened to meet the requirements of **CSE-1154-001-CAT E48 Version 2 Class F**.
- **41.2.** The equipment must withstand vibration and shock according to **IEC 61373**.

### 4.2. Power
- **4.2.1.** The LTU shall be powered from the locomotive’s 50-110V DC battery supply.
- **4.2.2.** The power supply shall be immunized against surges as stipulated in **IEC 60571** as a minimum.
- **4.2.3.** The LTU shall have a battery backup system to ensure continuous operation for at least 48 hours in the event of a primary power loss.
- **4.2.4.** The LTU shall implement a graceful shutdown mechanism to protect data and the filesystem in case of critical power loss.

### 4.3. Maintenance
- **4.3.1.** The system shall support remote firmware updates with rollback capability.
- **4.3.2.** The LTU shall support a diagnostic mode accessible via a secure local interface.
- **4LAG.3.** Hardware and software components shall be designed in a modular fashion for ease of repair or upgrade.

---

## 5. Security Requirements
- **5.1. Communication Security:** All remote server communications must be authenticated and encrypted over a secure channel (e.g., TLS).
- **5.2. Secure Access:** Local configuration and diagnostic access must require secure authentication.
- **5.3. Message Integrity:** Each LTU must securely process only the messages intended for its unique ID.
- **5.4. Private Network:** Cellular communication should be routed via a private APN where possible to isolate traffic from the public internet.
