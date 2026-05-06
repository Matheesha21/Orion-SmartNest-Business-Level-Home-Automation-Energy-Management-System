# Orion SmartNest – Step-by-Step Setup Guide

## Prerequisites

- Raspberry Pi 4B with power supply (5 V 3 A USB-C)
- 32 GB+ microSD card (A1/A2 rated for better I/O performance)
- PC or Mac with a microSD card reader
- Wi-Fi network (2.4 GHz)
- Smartphone with the **Home Assistant** companion app (iOS or Android)
- All hardware components listed in `docs/pin-mapping.md`

---

## Part 1 – Raspberry Pi Setup (Home Assistant OS)

### 1.1 Flash Home Assistant OS

1. Download the **Raspberry Pi Imager** from [rpi.imager](https://www.raspberrypi.com/software/).
2. Open the imager, select **Choose OS → Other specific-purpose OS → Home Assistants and home automation → Home Assistant**.
3. Select the `Raspberry Pi 4` image.
4. Select your microSD card.
5. Click **Write** and wait for the process to complete.

### 1.2 First Boot

1. Insert the microSD card into the Raspberry Pi.
2. Connect an Ethernet cable (strongly recommended for first setup) and the power supply.
3. Wait approximately **5–10 minutes** for Home Assistant to download and install.
4. Navigate to `http://homeassistant.local:8123` from any browser on the same network.
5. Complete the onboarding wizard (create admin account, set location, etc.).

### 1.3 Enable Advanced Mode

1. In HA, go to your **Profile** (bottom left).
2. Enable **Advanced Mode**.

---

## Part 2 – Install Required Add-ons

### 2.1 Mosquitto MQTT Broker

1. Go to **Settings → Add-ons → Add-on Store**.
2. Search for **Mosquitto broker** and install it.
3. After installation, go to the add-on configuration and add a user:
   ```yaml
   logins:
     - username: orion_mqtt
       password: <your_secure_password>
   ```
4. Start the add-on and enable **Start on boot** and **Watchdog**.

### 2.2 ESPHome Add-on

1. In Add-on Store, search for **ESPHome** and install it.
2. Start the add-on and open the Web UI.
3. Copy the YAML files from `esphome/` in this repository into the ESPHome dashboard.
4. Create a `secrets.yaml` file in the ESPHome add-on directory:
   ```yaml
   wifi_ssid: "YourWiFiSSID"
   wifi_password: "YourWiFiPassword"
   ap_password: "fallback_ap_password"
   api_encryption_key: "<32-byte base64 key>"   # generate with: openssl rand -base64 32
   ota_password: "your_ota_password"
   web_server_password: "your_web_password"
   ```
5. Compile and flash each device. For the first flash, use a USB cable. Subsequent updates can be done via OTA.

### 2.3 Tailscale Add-on

1. In Add-on Store, search for **Tailscale** and install it.
2. Start the add-on and click **Open Web UI** to authenticate with your Tailscale account.
3. Note the Tailscale IP address assigned to the Raspberry Pi (e.g., `100.x.y.z`).
4. Access Home Assistant remotely via `http://100.x.y.z:8123` from any device on your Tailscale network.

> **Tip:** Install Tailscale on your smartphone and laptop as well to access the dashboard from anywhere securely.

### 2.4 Studio Code Server (Optional)

Useful for editing YAML files directly from the browser.

1. Install **Studio Code Server** from the Add-on Store.
2. Set a password in the configuration and start it.

---

## Part 3 – Home Assistant Configuration

### 3.1 Copy Configuration Files

Place the contents of the `homeassistant/` directory into your Home Assistant `/config/` directory using one of:
- **Samba add-on** (file share over network)
- **Studio Code Server** (browser-based editor)
- **SSH & Terminal add-on** (command line)

The directory structure should be:
```
/config/
├── configuration.yaml
├── automations/
│   ├── lighting_automation.yaml
│   ├── climate_automation.yaml
│   ├── security_automation.yaml
│   └── energy_automation.yaml
├── scripts/
│   └── scripts.yaml
├── lovelace/
│   └── dashboard.yaml
└── packages/
    ├── mqtt.yaml
    ├── energy_monitor.yaml
    ├── notification.yaml
    └── input_helpers.yaml
```

### 3.2 Create secrets.yaml

Create `/config/secrets.yaml` (never commit this file to Git):

```yaml
mqtt_username: orion_mqtt
mqtt_password: <your_mqtt_password>
```

### 3.3 Validate and Restart

1. Go to **Settings → System → Check Configuration** and verify there are no errors.
2. Restart Home Assistant: **Settings → System → Restart**.

### 3.4 Add ESPHome Integration

1. Go to **Settings → Devices & Services → Add Integration**.
2. Search for **ESPHome**.
3. Enter the IP address of each ESP32 node and the API encryption key.
4. Repeat for all three nodes.

### 3.5 Set Up the Dashboard

1. Go to **Settings → Dashboards → Add Dashboard**.
2. Create a new dashboard named **Orion SmartNest** with a custom icon.
3. Edit the dashboard in YAML mode.
4. Paste the contents of `homeassistant/lovelace/dashboard.yaml`.

---

## Part 4 – Flashing ESP32 Devices

### 4.1 Prepare the Hardware

1. Wire each ESP32 node according to `docs/pin-mapping.md`.
2. Double-check all connections before powering on.

### 4.2 First Flash (USB)

1. Open the ESPHome Web UI on the Raspberry Pi.
2. Click **+ New Device** and enter the device name (e.g., `orion-living-room`).
3. Paste the corresponding YAML from `esphome/` (e.g., `living-room-node.yaml`).
4. Click **Install → Plug into this computer** if using ESPHome Web, or select the USB port.
5. Wait for compilation (~2–3 minutes) and flashing (~30 seconds).

### 4.3 Subsequent Updates (OTA)

Once the device is on the network:
1. Click **Install → Wirelessly**.
2. ESPHome will compile and upload the firmware over Wi-Fi.

---

## Part 5 – Mobile App Setup

1. Install the **Home Assistant** app from the App Store / Google Play.
2. Open the app and connect to `http://homeassistant.local:8123` on the local network.
3. Install **Tailscale** on the same phone.
4. Access the dashboard remotely via the Tailscale IP.

---

## Part 6 – Testing & Verification

### 6.1 Sensor Tests

| Test                          | Expected Result                                          |
|-------------------------------|----------------------------------------------------------|
| Wave hand in front of PIR     | Motion binary sensor turns `on` in dashboard             |
| Open monitored door/window    | Door/window sensor shows `open` in dashboard             |
| Heat DHT sensor with hand     | Temperature reading increases in dashboard               |
| Turn relay switch ON manually | Relay clicks and load (light/fan) activates              |

### 6.2 Automation Tests

| Test                                | Expected Result                                      |
|-------------------------------------|------------------------------------------------------|
| Enable Security Mode, open door     | Persistent notification appears in HA                |
| Motion detected, wait 10 min        | Light turns off automatically                        |
| Raise temperature above threshold   | Fan turns on automatically                           |
| Set Home State to Away              | All lights and fans turn off                         |

### 6.3 Remote Access Test

1. Disconnect your phone from home Wi-Fi.
2. Open Tailscale and confirm it is connected.
3. Open the HA app and navigate to the dashboard.
4. Toggle a switch – verify the device responds.

---

## Troubleshooting

| Issue                         | Solution                                                          |
|-------------------------------|-------------------------------------------------------------------|
| ESP32 not discovered in HA    | Check IP address in ESPHome dashboard; verify API key matches     |
| MQTT devices not updating     | Check Mosquitto add-on logs; verify username/password in secrets  |
| Dashboard shows unavailable   | Wait 30s after HA restart; check ESP32 is online (ping its IP)    |
| OTA flash fails               | Ensure ESP32 and Pi are on the same subnet; check firewall rules  |
| Tailscale not connecting      | Verify Tailscale is authenticated on both devices                 |
| DHT sensor shows NaN          | Check pull-up resistor (4.7 kΩ) and wire connections             |
| Relay not switching           | Verify relay is active-LOW; check `inverted: true` in YAML        |
