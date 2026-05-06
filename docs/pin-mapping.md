# Orion SmartNest – Pin Mapping & Wiring Reference

## ESP32 DevKit v1 – GPIO Usage Summary

> **Important notes:**
> - GPIO 34, 35, 36, 39 are **INPUT ONLY** – no internal pull-up/down.
> - GPIO 6–11 are connected to the flash chip – **do not use**.
> - Active-LOW relay boards: `HIGH` = relay OFF, `LOW` = relay ON.
> - The relay boards used are 5 V active-LOW 2-channel modules.

---

## Living Room Node (`orion-living-room`)

| GPIO | Direction | Component              | Notes                                    |
|------|-----------|------------------------|------------------------------------------|
| 21   | OUT (1-W) | DHT22 Data             | Add 4.7 kΩ pull-up to 3.3 V             |
| 25   | OUT (PWM) | Passive Buzzer (LEDC)  | 2 kHz via LEDC PWM                       |
| 26   | OUT       | Relay 1 – Light        | Active-LOW relay, inverted in YAML       |
| 27   | OUT       | Relay 2 – Fan          | Active-LOW relay, inverted in YAML       |
| 34   | IN        | PIR Motion Sensor OUT  | Input-only, no pull-up needed (PIR VCC→3.3V) |
| 35   | IN        | Magnetic Door Sensor   | Input-only, use external 10 kΩ pull-up  |

**Power:**
- ESP32 VIN ← 5 V USB power supply (2 A minimum)
- Relay module VCC ← 5 V (same supply)
- Relay module GND ← Common GND
- Relay IN1 ← GPIO 26, IN2 ← GPIO 27

---

## Bedroom Node (`orion-bedroom`)

| GPIO | Direction | Component              | Notes                                    |
|------|-----------|------------------------|------------------------------------------|
| 21   | OUT (1-W) | DHT11 Data             | Add 4.7 kΩ pull-up to 3.3 V             |
| 26   | OUT       | Relay 1 – Light        | Active-LOW relay, inverted in YAML       |
| 27   | OUT       | Relay 2 – Fan          | Active-LOW relay, inverted in YAML       |
| 34   | IN        | PIR Motion Sensor OUT  | Input-only                               |
| 35   | IN        | Magnetic Window Sensor | Input-only, use external 10 kΩ pull-up  |

---

## Kitchen Node (`orion-kitchen`)

| GPIO | Direction | Component              | Notes                                                |
|------|-----------|------------------------|------------------------------------------------------|
| 16   | IN (UART) | PZEM-004T TX → RX      | TTL 9600 baud UART; use 5V↔3.3V level shifter        |
| 17   | OUT (UART)| PZEM-004T RX ← TX      | TTL 9600 baud UART; use 5V↔3.3V level shifter        |
| 21   | OUT (1-W) | DHT22 Data             | Add 4.7 kΩ pull-up to 3.3 V                         |
| 26   | OUT       | Relay 1 – Light        | Active-LOW relay                                     |
| 27   | OUT       | Relay 2 – Exhaust Fan  | Active-LOW relay                                     |
| 34   | IN        | PIR Motion Sensor OUT  | Input-only                                           |
| 35   | IN        | Magnetic Door Sensor   | Input-only, use external 10 kΩ pull-up               |

> **PZEM-004T Warning:** The PZEM-004T module operates at 5 V logic.
> Always use a bi-directional 3.3 V / 5 V level shifter on the UART lines
> to protect the ESP32's 3.3 V GPIO pins.

---

## Wiring Diagrams (Text Schematic)

### PIR Sensor Wiring
```
PIR Sensor
  VCC  ──────────── 3.3 V (ESP32)
  GND  ──────────── GND  (ESP32)
  OUT  ──────────── GPIO 34 (input-only)
```

### DHT11 / DHT22 Wiring
```
DHT Sensor
  VCC  ──────────── 3.3 V (ESP32)
  GND  ──────────── GND  (ESP32)
  DATA ──┬───────── GPIO 21 (ESP32)
         └── 4.7 kΩ pull-up resistor ── 3.3 V
```

### Active-LOW 2-Channel Relay Module Wiring
```
Relay Module
  VCC  ──────────── 5 V  (external supply)
  GND  ──────────── GND  (common with ESP32)
  IN1  ──────────── GPIO 26 (ESP32)
  IN2  ──────────── GPIO 27 (ESP32)

Relay Output (NO / COM)
  COM  ──── Live (AC)
  NO   ──── Load (light / fan)
  Neutral ── Neutral (AC)
```

### Magnetic Door/Window Sensor Wiring
```
Magnetic Sensor (reed switch – normally open or normally closed)
  Pin A ──────────── GPIO 35 (input-only)
  Pin B ──────────── GND

External pull-up:
  GPIO 35 ──── 10 kΩ ──── 3.3 V
  (When magnet present: circuit OPEN → pin HIGH; when separated: pin LOW)
  Note: YAML uses INPUT_PULLUP and inverted: true for normally-closed type.
```

### Passive Buzzer Wiring (Living Room only)
```
Buzzer
  + ──────────── GPIO 25 (PWM via LEDC)
  - ──────────── GND

Optional: add NPN transistor (e.g. 2N2222) for higher drive current:
  GPIO 25 ── 1 kΩ ── Base
  Collector ── Buzzer(+) ── 3.3 V
  Emitter ── GND
  Buzzer(-) ── Collector
```

### PZEM-004T Energy Meter Wiring (Kitchen only)
```
PZEM-004T (5V side)           Level Shifter            ESP32 (3.3V side)
  TX ─────────────── HV1 ─── LV1 ─────────── GPIO 16 (RX)
  RX ─────────────── HV2 ─── LV2 ─────────── GPIO 17 (TX)
  GND ─────────────────────────────────────── GND
  5V ──────────────────────────────────────── 5V

AC Live ────── PZEM-004T AC Input (Line)
AC Neutral ─── PZEM-004T AC Input (Neutral)
Load Live ──── PZEM-004T AC Output (Line) ── Load
Load Neutral ─ AC Neutral ──────────────── Load

CT Clamp: clip around the Live conductor of the monitored circuit.
```

---

## Component Bill of Materials (BOM)

| # | Component                        | Qty | Notes                                |
|---|----------------------------------|-----|--------------------------------------|
| 1 | ESP32 DevKit v1                  | 3   | One per room                         |
| 2 | DHT22 temperature/humidity       | 2   | Living room + kitchen                |
| 3 | DHT11 temperature/humidity       | 1   | Bedroom (lower cost)                 |
| 4 | HC-SR501 PIR motion sensor       | 3   | One per room                         |
| 5 | 2-channel 5 V active-LOW relay   | 3   | One per room, 10 A rated             |
| 6 | Magnetic door/window sensor      | 3   | Normally-closed type recommended     |
| 7 | Passive buzzer 5 V               | 1   | Living room security alert           |
| 8 | PZEM-004T energy meter           | 1   | Kitchen circuit monitoring           |
| 9 | Bi-directional level shifter     | 1   | For PZEM-004T UART                   |
|10 | 4.7 kΩ resistors                 | 3   | DHT pull-up                          |
|11 | 10 kΩ resistors                  | 3   | Door sensor pull-up                  |
|12 | 5 V 2 A USB power adapters       | 3   | One per ESP32 node                   |
|13 | Raspberry Pi 4B (2 GB+)         | 1   | Central server                       |
|14 | 32 GB+ microSD card (A2 class)   | 1   | Home Assistant OS                    |
|15 | Ethernet cable                   | 1   | Recommended: wired LAN for Pi        |
