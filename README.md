# ESP32 Smart Irrigation System 

An IoT-based smart irrigation system designed to monitor soil moisture across multiple zones and automate water pump control using an ESP32.

The system uses **nine soil moisture sensors**, spatial analysis, and a web-based dashboard to help monitor soil conditions and manage irrigation efficiently.

##  Features

- **Multi-zone monitoring:** Reads soil moisture from nine sensor locations.
- **Automated irrigation:** Controls the water pump through a relay.
- **Live web dashboard:** Monitors soil moisture and system status.
- **Spatial analysis:** Supports moisture mapping and analysis of sensor relationships.
- **Configurable settings:** Uses a configuration file to manage irrigation parameters.
- **System monitoring:** Includes operating states and diagnostic documentation.

##  Hardware Components

- ESP32 development board
- Capacitive soil moisture sensors (9)
- CD4051 multiplexer
- Relay module
- Water pump and suitable power supply
- Connecting wires and supporting circuit components

*Check the wiring diagram and deployment guide for the intended hardware connections before assembling the system.*

##  System Overview

1. Soil moisture sensors collect readings from different locations.
2. The ESP32 reads the sensors and processes the measurements.
3. The system evaluates soil conditions using configured parameters.
4. The pump relay controls irrigation when required.
5. The web dashboard displays sensor readings and system information.

##  Project Documentation

This repository includes guides and supporting files for understanding, configuring, testing, and deploying the system.

- **Deployment Guide:** Hardware setup, software upload, installation, calibration, and troubleshooting.
- **Power System Guide:** Power-related design and operating considerations.
- **Quick Reference:** A concise project reference.
- **Phase Documentation:** Development notes, improvements, and implementation summaries.
- **Diagrams:** System and software workflow diagrams.
- **Configuration:** Project configuration settings.

##  Getting Started

1. Review the deployment guide and wiring diagrams.
2. Install the Arduino IDE and the required ESP32 board support.
3. Review the firmware and required libraries.
4. Configure your local Wi-Fi credentials securely.
5. Check the sensor connections and configuration settings.
6. Upload the firmware to the ESP32 and test the system.
7. Calibrate the moisture sensors and verify pump operation before field use.

Refer to the project's deployment and testing documentation for detailed instructions.

##  Security and Safety

- Do not publish Wi-Fi passwords, private network credentials, or other secrets.
- Keep local credentials out of public source code.
- Test relay and pump operation safely before connecting the system to a live irrigation setup.
- Use an appropriate power supply and suitable electrical protection.

##  Project Goal

To explore IoT-based irrigation, multi-point soil moisture monitoring, spatial analysis, and automated water management using an ESP32.

##  Project Status

This repository contains project firmware, configuration-related files, diagrams, and development documentation. Refer to the individual guides for implementation details and testing procedures.
