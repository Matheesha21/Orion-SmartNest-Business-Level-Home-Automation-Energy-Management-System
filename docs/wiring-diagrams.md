# Orion SmartNest – Wiring Diagrams

This document provides visual wiring schematics for all three ESP32 room nodes.
Refer to `pin-mapping.md` for the GPIO assignments and BOM.

---

## Living Room Node Wiring Diagram

```
                    ┌─────────────────────────────────────────────┐
                    │            ESP32 DevKit v1                   │
                    │                                              │
  5V USB ──── VIN   │ VIN                                 GND ──── GND ──┐
              GND   │                                              │      │
                    │  GPIO21 ──── DHT22 DATA ──── 4.7kΩ ── 3.3V │      │
                    │                                              │      │
                    │  GPIO25 ──── Buzzer (+)                      │      │
                    │                                              │      │
                    │  GPIO26 ──── Relay IN1 ── [Relay 1] ──── Light │  │
                    │                                              │      │
                    │  GPIO27 ──── Relay IN2 ── [Relay 2] ──── Fan  │   │
                    │                                              │      │
                    │  GPIO34 ──── PIR OUT                         │      │
                    │                                              │      │
                    │  GPIO35 ──── Door Sensor A ── 10kΩ ── 3.3V │      │
                    └─────────────────────────────────────────────┘      │
                                                                          │
  DHT22 GND ──────────────────────────────────────────────────────────────┤
  DHT22 VCC ──── 3.3V                                                      │
  PIR GND ────────────────────────────────────────────────────────────────┤
  PIR VCC ──── 3.3V                                                        │
  Door Sensor B ──────────────────────────────────────────────────────────┤
  Relay GND ──────────────────────────────────────────────────────────────┤
  Relay VCC ──── 5V                                                        │
  Buzzer (-)  ─────────────────────────────────────────────────────────────┘

Relay Output (Mains – do with care):
  Relay 1 COM ──── Mains Live ────── Light
  Relay 1 NO  ──── Light (return to Neutral)
  Relay 2 COM ──── Mains Live ────── Fan
  Relay 2 NO  ──── Fan (return to Neutral)
```

---

## Bedroom Node Wiring Diagram

```
                    ┌─────────────────────────────────────────────┐
                    │            ESP32 DevKit v1                   │
                    │                                              │
  5V USB ──── VIN   │ VIN                                 GND ──── GND ──┐
              GND   │                                              │      │
                    │  GPIO21 ──── DHT11 DATA ──── 4.7kΩ ── 3.3V │      │
                    │                                              │      │
                    │  GPIO26 ──── Relay IN1 ── [Relay 1] ─── Light │    │
                    │                                              │      │
                    │  GPIO27 ──── Relay IN2 ── [Relay 2] ─── Fan   │   │
                    │                                              │      │
                    │  GPIO34 ──── PIR OUT                         │      │
                    │                                              │      │
                    │  GPIO35 ──── Window Sensor A ── 10kΩ ─ 3.3V │      │
                    └─────────────────────────────────────────────┘      │
                                                                          │
  DHT11 GND ──────────────────────────────────────────────────────────────┤
  DHT11 VCC ──── 3.3V                                                      │
  PIR GND ────────────────────────────────────────────────────────────────┤
  PIR VCC ──── 3.3V                                                        │
  Window Sensor B ────────────────────────────────────────────────────────┤
  Relay GND ──────────────────────────────────────────────────────────────┤
  Relay VCC ──── 5V                                                        │
                                                                           └─ Common GND
```

---

## Kitchen Node Wiring Diagram

```
                    ┌──────────────────────────────────────────────────────┐
                    │                   ESP32 DevKit v1                    │
                    │                                                      │
  5V USB ──── VIN   │ VIN                                      GND ─────── GND ──┐
              GND   │                                                      │      │
                    │  GPIO16 (RX) ── LV1 ─ HV1 ── PZEM TX               │      │
                    │  GPIO17 (TX) ── LV2 ─ HV2 ── PZEM RX               │      │
                    │           Level Shifter (3.3V ↔ 5V)                │      │
                    │                                                      │      │
                    │  GPIO21 ──── DHT22 DATA ──── 4.7kΩ ──── 3.3V      │      │
                    │                                                      │      │
                    │  GPIO26 ──── Relay IN1 ── [Relay 1] ─── Kitchen Light │   │
                    │                                                      │      │
                    │  GPIO27 ──── Relay IN2 ── [Relay 2] ─── Exhaust Fan  │   │
                    │                                                      │      │
                    │  GPIO34 ──── PIR OUT                                 │      │
                    │                                                      │      │
                    │  GPIO35 ──── Door Sensor A ── 10kΩ ──── 3.3V       │      │
                    └──────────────────────────────────────────────────────┘      │
                                                                                   │
  Level Shifter:                                                                    │
    LV  ── 3.3V (ESP32)                                                            │
    HV  ── 5V (external / PZEM)                                                    │
    GND ───────────────────────────────────────────────────────────────────────────┤
                                                                                   │
  PZEM-004T:                                                                       │
    GND ───────────────────────────────────────────────────────────────────────────┤
    5V  ──── 5V (from ESP32 or separate supply)                                    │
    AC IN (Line)    ──── Mains Live                                                │
    AC IN (Neutral) ──── Mains Neutral                                             │
    AC OUT (Line)   ──── Load Live                                                 │
    CT Clamp        ──── Clipped around the Live conductor                         │
                                                                                   └─ Common GND
```

---

## Raspberry Pi 4B Connection Diagram

```
                    ┌─────────────────────────────────┐
                    │        Raspberry Pi 4B           │
                    │                                  │
  Power Supply 5V ──┤ USB-C Power                      │
  Ethernet Cable ───┤ RJ45 LAN ──── Home Router        │
                    │                                  │
                    │  Wi-Fi 2.4GHz ──── (for HA UI)   │
                    │  (primary: use Ethernet)         │
                    │                                  │
                    │  Running:                        │
                    │  • Home Assistant OS             │
                    │  • Mosquitto MQTT Broker         │
                    │  • ESPHome Add-on                │
                    │  • Tailscale VPN Add-on          │
                    └─────────────────────────────────┘
                              │ LAN (Wi-Fi 2.4GHz)
          ┌───────────────────┼───────────────────┐
          ▼                   ▼                   ▼
   ESP32 Living Room    ESP32 Bedroom       ESP32 Kitchen
   192.168.1.20         192.168.1.21        192.168.1.22
```

---

## Safety Notes

> ⚠️ **MAINS VOLTAGE WARNING**
>
> Relay modules control mains voltage (110 V or 230 V AC).
> Working with mains voltage is dangerous and can be fatal.
>
> - Always isolate the mains circuit before wiring.
> - Use an insulated enclosure for any relay mounted to mains wiring.
> - If you are not qualified to work with mains electricity, use a certified
>   electrician for the mains-side wiring.
> - The PZEM-004T must be connected to mains – exercise extreme caution.
> - Prototype and test low-voltage sections (sensors, ESP32) separately
>   before connecting to mains.
