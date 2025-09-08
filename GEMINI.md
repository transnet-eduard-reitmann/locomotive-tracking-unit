# Gemini Project Context: Locomotive Tracking Unit

## Project Overview

This repository contains the full-stack implementation of a GPS tracking system for locomotive fleet management. The system is designed for the South African railway network, providing real-time location data, modular communication options, and a comprehensive management dashboard.

**Core Technologies:**

*   **Hardware:** LILYGO T-A7670G R2 Q425 (ESP32)
*   **Firmware:** C++ with PlatformIO
*   **Backend:** .NET Core, C#, MS SQL Server
*   **Frontend:** React.js, Redux, Material-UI
*   **Communication:** MQTT over TLS, Cellular (4G/3G/2G), with optional LoRa and Satellite modules.

**Architecture:**

The system follows a modular, microservice-oriented architecture:

*   **Firmware:** An ESP32-based application captures GPS data and transmits it over the most efficient communication channel available. It supports Over-the-Air (OTA) updates.
*   **Backend:** A .NET Core application provides a RESTful API for device management, data storage, and real-time communication via SignalR. It uses an MS SQL Server database with spatial extensions.
*   **Frontend:** A React.js single-page application (SPA) serves as a dashboard for real-time fleet monitoring, data analysis, and system administration.

## Building and Running

### Firmware

*   **Prerequisites:** PlatformIO CLI
*   **Build:**
    ```bash
    cd firmware/core
    pio run
    ```
*   **Upload:**
    ```bash
    # Via serial
    pio run --target upload --upload-port <your_port>

    # Via OTA
    # pio run --target upload --upload-protocol espota --upload-port <device_ip>
    ```

### Backend

*   **Prerequisites:** .NET 8 SDK, Docker (for MQTT broker)
*   **Run MQTT Broker:**
    ```bash
    docker run -it -p 1883:1883 -p 9001:9001 eclipse-mosquitto
    ```
*   **Run API & Services:**
    ```bash
    cd backend/api
    dotnet run
    ```
*   **Run Tests:**
    ```bash
    dotnet test
    ```

### Frontend

*   **Prerequisites:** Node.js, npm
*   **Install & Run:**
    ```bash
    cd frontend/dashboard
    npm install
    npm run dev
    ```
*   **Build for Production:**
    ```bash
    npm run build
    ```

## Development Conventions

*   **Firmware:**
    *   Code is structured into `core`, `modules`, and `config` directories within the `firmware` directory.
    *   PlatformIO is the standard build and dependency management tool.
    *   OTA updates are a key feature, with partition schemes and libraries configured in `platformio.ini`.
*   **Backend:**
    *   Follows a standard .NET Core API project structure within the `backend` directory.
    *   Uses Entity Framework Core for database migrations.
    *   Services are separated into `api`, `services`, and `database` projects.
    *   xUnit is the designated framework for unit testing.
*   **Frontend:**
    *   Built with Vite and React in the `frontend/dashboard` directory.
    *   State is managed using Redux Toolkit.
    *   Components are styled with Material-UI.
    *   API interaction is handled by `axios`, and real-time updates by `@microsoft/signalr`.
*   **Git:**
    *   The project includes a `TODO.md` which is actively maintained and provides a roadmap.
    *   The `.archive` directory contains historical research and previous project structures, preserved for context after a major refactoring.