# Orion SmartNest – System Architecture

## High-Level Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          INTERNET / WAN                                  │
│                    (Secure access via Tailscale VPN)                     │
└────────────────────────────┬────────────────────────────────────────────┘
                             │ Encrypted Tailscale tunnel
                             ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              RASPBERRY PI 4B  –  Central Home Automation Server          │
│                                                                          │
│  ┌─────────────────────┐   ┌──────────────────┐   ┌──────────────────┐  │
│  │   Home Assistant OS  │   │  Mosquitto MQTT  │   │   Tailscale VPN  │  │
│  │  (Port 8123 – HTTP) │   │  Broker :1883    │   │  (WireGuard UDP) │  │
│  └────────┬────────────┘   └────────┬─────────┘   └──────────────────┘  │
│           │  ESPHome Native API      │  MQTT (optional topics)            │
│           │  (port 6053, encrypted)  │                                    │
└───────────┼─────────────────────────┼────────────────────────────────────┘
            │                         │
            │  Wi-Fi (2.4 GHz)        │
            ▼                         ▼
┌───────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐
│  ESP32 – Living Room  │  │  ESP32 – Bedroom     │  │  ESP32 – Kitchen     │
│                       │  │                      │  │                      │
│ • PIR Motion Sensor   │  │ • PIR Motion Sensor  │  │ • PIR Motion Sensor  │
│ • DHT22 Temp/Humidity │  │ • DHT11 Temp/Humidity│  │ • DHT22 Temp/Humidity│
│ • Relay – Light       │  │ • Relay – Light      │  │ • Relay – Light      │
│ • Relay – Fan         │  │ • Relay – Fan        │  │ • Relay – Exhaust Fan│
│ • Door Sensor         │  │ • Window Sensor      │  │ • Door Sensor        │
│ • Passive Buzzer      │  │                      │  │ • PZEM-004T Energy   │
└───────────────────────┘  └──────────────────────┘  └──────────────────────┘
```

## Communication Stack

| Layer              | Technology                          | Purpose                                      |
|--------------------|-------------------------------------|----------------------------------------------|
| Remote Access      | Tailscale (WireGuard VPN)           | Secure remote access without port forwarding |
| Local API          | ESPHome Native API (encrypted gRPC) | Primary ESP32 ↔ HA communication             |
| Fallback Messaging | MQTT (Mosquitto)                    | Third-party device and custom topic support  |
| Firmware           | ESPHome (Arduino framework)         | ESP32 device firmware and sensor reading     |
| Automation Engine  | Home Assistant                      | All rules, scripts, and state management     |
| Dashboard          | Home Assistant Lovelace             | Web and mobile UI                            |

## Data Flow

```
Sensor Event (PIR triggers)
        │
        ▼
ESP32 reads GPIO
        │
        ▼
ESPHome firmware processes event
        │  Native API / MQTT
        ▼
Home Assistant receives state change
        │
        ▼
Automation engine evaluates trigger, condition, and action
        │
        ▼
Action executed: switch relay, send notification, log event
        │
        ▼
Dashboard updated in real time (WebSocket push)
```

## Network Topology

```
Home Router (192.168.1.0/24)
  ├── Raspberry Pi 4B        192.168.1.10  (static)
  ├── ESP32 Living Room      192.168.1.20  (static DHCP lease)
  ├── ESP32 Bedroom          192.168.1.21  (static DHCP lease)
  └── ESP32 Kitchen          192.168.1.22  (static DHCP lease)

Tailscale Network (100.x.y.z)
  └── Raspberry Pi 4B        100.x.y.10   (Tailscale IP – access HA remotely)
```

## Security Architecture

```
External Device (phone / laptop)
         │
         │  WireGuard encrypted tunnel (Tailscale)
         ▼
Tailscale relay / direct peer-to-peer
         │
         ▼
Raspberry Pi 4B – Home Assistant (port 8123)
         │
         │  All internal traffic on LAN – no port forwarding required
         ▼
ESP32 nodes – ESPHome API (encrypted key per device)
```

## Component Responsibilities

### Raspberry Pi 4B
- Runs **Home Assistant OS** (supervised installation).
- Hosts **Mosquitto MQTT broker** (add-on).
- Hosts **ESPHome add-on** for firmware compilation and OTA updates.
- Hosts **Tailscale add-on** for secure remote access.
- Persistent 32 GB+ microSD or SSD storage.

### ESP32 DevKit v1 Nodes
- Low-power always-on microcontroller.
- Reads sensors and controls relays via GPIO.
- Communicates with HA via encrypted ESPHome API over LAN.
- Supports OTA (over-the-air) firmware updates.
- Hosts a basic web server for local diagnostics.

### ESPHome
- Generates C++ firmware from YAML configuration.
- Compiles and flashes directly from the Raspberry Pi web UI.
- Manages Wi-Fi reconnection, watchdog, and OTA.

### Tailscale
- Creates a private WireGuard mesh network.
- No router port forwarding required.
- Supports MagicDNS for friendly hostnames.
- Works across NAT, firewalls, and mobile networks.
