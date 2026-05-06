# 🏠 Orion SmartNest
### Business-Level Smart Home Automation & Energy Management System

> A production-quality IoT solution built on Raspberry Pi 4B, ESP32 microcontrollers,
> Home Assistant, ESPHome, and Tailscale – designed as a university final-year project.

[![Home Assistant](https://img.shields.io/badge/Home%20Assistant-2024.x-blue?logo=home-assistant)](https://www.home-assistant.io/)
[![ESPHome](https://img.shields.io/badge/ESPHome-2024.x-green?logo=esphome)](https://esphome.io/)
[![Tailscale](https://img.shields.io/badge/Tailscale-VPN-purple?logo=tailscale)](https://tailscale.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Features](#-features)
- [Repository Structure](#-repository-structure)
- [Hardware Requirements](#-hardware-requirements)
- [Quick Start](#-quick-start)
- [ESP32 Room Nodes](#-esp32-room-nodes)
- [Home Assistant Configuration](#-home-assistant-configuration)
- [Dashboard](#-dashboard)
- [Security](#-security)
- [Energy Management](#-energy-management)
- [Remote Access (Tailscale)](#-remote-access-tailscale)
- [Documentation](#-documentation)
- [License](#-license)

---

## 🌟 Overview

**Orion SmartNest** is a modular, scalable, and secure business-level home automation and energy management system. It combines the power of **Home Assistant** as the central automation engine with **ESP32** distributed sensor nodes flashed via **ESPHome**, and uses **Tailscale VPN** for zero-configuration secure remote access.

The system is organized as a real commercial IoT solution with:
- Modular YAML firmware per room (easy to add new rooms)
- Encrypted ESPHome API communication
- Comprehensive automation rules for lighting, climate, security, and energy
- A modern, multi-view Lovelace dashboard accessible from mobile and desktop
- Structured documentation for hardware setup, wiring, and deployment

---

## 🏗️ System Architecture

```
Internet / WAN
      │  Tailscale WireGuard VPN
      ▼
Raspberry Pi 4B  ←──── Home Assistant OS
      │  ESPHome Native API (encrypted)
      │  MQTT (Mosquitto broker)
      ├──── ESP32 Living Room Node
      ├──── ESP32 Bedroom Node
      └──── ESP32 Kitchen Node
```

See [`docs/system-architecture.md`](docs/system-architecture.md) for the full architecture diagram and communication stack.

---

## ✨ Features

### 💡 Smart Lighting Automation
- PIR motion sensor detects occupancy in each room
- Automatic light ON/OFF based on motion
- Configurable auto-off timer (1–60 minutes, adjustable from dashboard)
- Manual override available at any time via dashboard

### 🌡️ Smart Temperature & Fan Control
- DHT11/DHT22 sensors for temperature and humidity in each room
- Automatic fan activation when temperature exceeds configurable threshold
- Dual-threshold hysteresis (ON threshold / OFF threshold) to prevent rapid cycling
- Live environmental data displayed on dashboard

### 🔒 Security Monitoring
- Magnetic door/window sensors on all entry points
- Motion detection alerts during security mode
- Security buzzer / siren integration (living room)
- Dashboard security status indicators
- Arm / disarm security mode from dashboard or scripts

### ⚡ Energy Management
- Real-time power monitoring with PZEM-004T energy meter (kitchen)
- Alerts for appliances left on too long
- Away Mode: automatically turn off all devices when leaving
- Night Mode: optimise thresholds for sleep comfort
- Daily / monthly / yearly energy usage tracking (utility meter)
- Estimated cost display (configurable per-kWh rate)

### 🌐 Remote Monitoring & Control
- Secure remote access via Tailscale (no port forwarding required)
- Real-time sensor updates via WebSocket push
- Mobile-responsive dashboard
- Multi-room, multi-view organised interface

---

## 📁 Repository Structure

```
Orion-SmartNest/
├── esphome/
│   ├── common/
│   │   └── base-config.yaml          # Shared Wi-Fi, API, OTA, diagnostics
│   ├── living-room-node.yaml         # Living room ESP32 firmware
│   ├── bedroom-node.yaml             # Bedroom ESP32 firmware
│   └── kitchen-node.yaml             # Kitchen ESP32 firmware (+ PZEM energy)
│
├── homeassistant/
│   ├── configuration.yaml            # Main HA configuration entry point
│   ├── automations/
│   │   ├── lighting_automation.yaml  # Motion-based light control
│   │   ├── climate_automation.yaml   # Temperature-based fan control
│   │   ├── security_automation.yaml  # Door/motion security alerts
│   │   └── energy_automation.yaml   # Energy saving rules and alerts
│   ├── scripts/
│   │   └── scripts.yaml             # Good Night, Good Morning, Emergency scripts
│   ├── lovelace/
│   │   └── dashboard.yaml           # Full Lovelace UI (6 views)
│   └── packages/
│       ├── mqtt.yaml                 # MQTT sensors and switches
│       ├── energy_monitor.yaml       # Template sensors, utility meters
│       ├── notification.yaml         # Notification channels
│       └── input_helpers.yaml        # input_boolean, input_number, input_select
│
└── docs/
    ├── system-architecture.md        # Architecture diagrams and data flow
    ├── setup-guide.md                # Step-by-step deployment guide
    ├── wiring-diagrams.md            # ASCII wiring schematics per room
    └── pin-mapping.md                # GPIO table and BOM
```

---

## 🔧 Hardware Requirements

| Component                    | Qty | Role                                       |
|------------------------------|-----|--------------------------------------------|
| Raspberry Pi 4B (2 GB+)      | 1   | Central server – Home Assistant OS         |
| 32 GB A2 microSD card        | 1   | OS storage                                 |
| ESP32 DevKit v1              | 3   | Smart room nodes (living room, bedroom, kitchen) |
| DHT22 temp/humidity sensor   | 2   | Living room + kitchen                      |
| DHT11 temp/humidity sensor   | 1   | Bedroom                                    |
| HC-SR501 PIR motion sensor   | 3   | One per room                               |
| 2-ch 5 V relay module        | 3   | Light and fan/appliance control            |
| Magnetic door/window sensor  | 3   | Entry point monitoring                     |
| Passive buzzer               | 1   | Living room security alert                 |
| PZEM-004T energy meter       | 1   | Kitchen power monitoring                   |
| Bi-directional level shifter | 1   | PZEM-004T UART level conversion            |

Full BOM with resistor values: [`docs/pin-mapping.md`](docs/pin-mapping.md)

---

## 🚀 Quick Start

### 1. Flash Home Assistant OS
Download and flash **Home Assistant OS** for Raspberry Pi 4 using the [Raspberry Pi Imager](https://www.raspberrypi.com/software/). Boot and complete onboarding at `http://homeassistant.local:8123`.

### 2. Install Add-ons
From the Home Assistant Add-on Store, install:
- **Mosquitto MQTT Broker** – local MQTT
- **ESPHome** – firmware builder and OTA manager
- **Tailscale** – secure remote access
- **Studio Code Server** (optional) – browser-based YAML editor

### 3. Deploy Configuration
Copy the `homeassistant/` directory contents to `/config/` on your Home Assistant instance.

### 4. Create Secrets
Create `/config/secrets.yaml` with your credentials (not committed to this repo):
```yaml
mqtt_username: orion_mqtt
mqtt_password: your_secure_password
```

Create `esphome/secrets.yaml`:
```yaml
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"
ap_password: "fallback_password"
api_encryption_key: "<base64 key from: openssl rand -base64 32>"
ota_password: "your_ota_password"
web_server_password: "your_web_password"
```

### 5. Flash ESP32 Nodes
Open the ESPHome dashboard, paste each YAML file, and flash over USB for the first time. Subsequent updates are wireless (OTA).

### 6. Wire the Hardware
Follow [`docs/wiring-diagrams.md`](docs/wiring-diagrams.md) and [`docs/pin-mapping.md`](docs/pin-mapping.md).

For the complete step-by-step guide, see [`docs/setup-guide.md`](docs/setup-guide.md).

---

## 📡 ESP32 Room Nodes

### Living Room (`esphome/living-room-node.yaml`)
| Sensor / Actuator     | GPIO | Description                               |
|-----------------------|------|-------------------------------------------|
| DHT22                 | 21   | Temperature & humidity                    |
| PIR Motion            | 34   | Occupancy detection                       |
| Relay – Light         | 26   | Active-LOW relay, ceiling light control   |
| Relay – Fan           | 27   | Active-LOW relay, ceiling fan control     |
| Magnetic Door Sensor  | 35   | Door open/close detection                 |
| Passive Buzzer        | 25   | Security alert siren (PWM via LEDC)       |

### Bedroom (`esphome/bedroom-node.yaml`)
| Sensor / Actuator     | GPIO | Description                               |
|-----------------------|------|-------------------------------------------|
| DHT11                 | 21   | Temperature & humidity                    |
| PIR Motion            | 34   | Occupancy detection                       |
| Relay – Light         | 26   | Bedroom light control                     |
| Relay – Fan           | 27   | Bedroom fan control                       |
| Magnetic Window Sensor| 35   | Window open/close detection               |

### Kitchen (`esphome/kitchen-node.yaml`)
| Sensor / Actuator     | GPIO | Description                               |
|-----------------------|------|-------------------------------------------|
| DHT22                 | 21   | Temperature & humidity                    |
| PIR Motion            | 34   | Occupancy detection                       |
| Relay – Light         | 26   | Kitchen light control                     |
| Relay – Exhaust Fan   | 27   | Exhaust fan control                       |
| Magnetic Door Sensor  | 35   | Door open/close detection                 |
| PZEM-004T (TX/RX)     | 16/17| Real-time power, voltage, current, energy |

---

## 🤖 Home Assistant Configuration

### Automations

| File                       | Automations                                                     |
|----------------------------|-----------------------------------------------------------------|
| `lighting_automation.yaml` | Light ON on motion, light OFF after inactivity (all 3 rooms)   |
| `climate_automation.yaml`  | Fan ON/OFF by temperature threshold (all 3 rooms)              |
| `security_automation.yaml` | Door/window/motion alerts in security mode, arm/disarm notify  |
| `energy_automation.yaml`   | Appliance-on-too-long alerts, Away Mode, high power alert       |

### Scripts

| Script Name          | Description                                             |
|----------------------|---------------------------------------------------------|
| `good_night`         | Sets Night mode, arms security, turns off common lights |
| `good_morning`       | Disarms security, restores defaults, sets Home mode     |
| `all_lights_off`     | Turns off all room lights                               |
| `all_fans_off`       | Turns off all fans/exhaust fans                         |
| `emergency_all_on`   | Emergency – all lights on and alarm sounding            |

---

## 📊 Dashboard

The Lovelace dashboard (`homeassistant/lovelace/dashboard.yaml`) has 6 views:

| View         | Content                                                          |
|--------------|------------------------------------------------------------------|
| **Overview** | Home state, quick actions (Good Night/Morning), security toggle, energy summary |
| **Living Room** | Sensor glance, device controls, diagnostic info              |
| **Bedroom**  | Sensor glance, device controls, diagnostic info                  |
| **Kitchen**  | Sensor glance, device controls, PZEM-004T energy panel          |
| **Security** | Security mode toggle, all entry points, motion sensors, history |
| **Energy**   | Real-time power, usage stats, history graph, threshold settings  |

---

## 🔐 Security

- **ESPHome API**: Each device uses a unique 32-byte encrypted API key (stored in `secrets.yaml`, never committed to Git).
- **OTA**: OTA updates require a password.
- **Tailscale**: WireGuard-based VPN mesh; no ports exposed to the public internet.
- **MQTT**: Password-protected Mosquitto broker; only local LAN access.
- **Secrets Management**: All sensitive values are kept in `secrets.yaml` files excluded by `.gitignore`.

---

## ⚡ Energy Management

The kitchen node includes a **PZEM-004T** energy meter that provides:
- Real-time **voltage**, **current**, **power**, **energy**, **frequency**, and **power factor**
- Persistent energy accumulation (kWh) that survives reboots
- Daily / monthly / yearly energy tracking via Home Assistant **utility_meter**
- Estimated daily cost in dollars based on a configurable per-kWh rate
- High-power alert (>2 kW for >5 minutes)

---

## 🌐 Remote Access (Tailscale)

Tailscale creates an encrypted peer-to-peer WireGuard mesh network.

1. Create a free account at [tailscale.com](https://tailscale.com/).
2. Install the Tailscale add-on in Home Assistant.
3. Authenticate the Raspberry Pi to your Tailscale account.
4. Install Tailscale on your phone/laptop.
5. Access Home Assistant from anywhere via `http://<tailscale-ip>:8123`.

No router configuration, no port forwarding, no dynamic DNS required.

---

## 📖 Documentation

| Document                                              | Description                               |
|-------------------------------------------------------|-------------------------------------------|
| [`docs/system-architecture.md`](docs/system-architecture.md) | Full architecture diagram, data flow, network topology |
| [`docs/setup-guide.md`](docs/setup-guide.md)         | Complete step-by-step deployment guide    |
| [`docs/wiring-diagrams.md`](docs/wiring-diagrams.md) | ASCII wiring schematics for all 3 nodes   |
| [`docs/pin-mapping.md`](docs/pin-mapping.md)         | GPIO tables, BOM, component notes         |

---

## 📜 License

This project is licensed under the **MIT License**.

---

*Orion SmartNest – Built for the future of intelligent living.*
