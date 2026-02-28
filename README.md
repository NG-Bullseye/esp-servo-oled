# esp-servo-oled

ESPHome project: MC-410 servo + SSD1306 OLED display on ESP32 NodeMCU, controlled via Home Assistant.

---

## Hardware

**NodeMCU ESP32**

### Servo — MC-410 Standard Modelcraft

| Cable | Color | ESP32 Pin |
|-------|-------|-----------|
| GND | Brown | GND |
| VCC | Red | VIN (5V from USB) |
| Signal | Orange | GPIO 13 |

> ⚠️ Servo requires 5V — use VIN, not 3.3V. Signal at 3.3V logic level works fine with MC-410.

### OLED Display — SSD1306 128x64 I2C

| Cable | Color | ESP32 Pin |
|-------|-------|-----------|
| VCC | Red | 3.3V |
| GND | Blue | GND |
| SCL | Orange | GPIO 22 |
| SDA | White | GPIO 21 |

---

## Wiring Diagram

```
ESP32 NodeMCU
┌─────────────────────┐
│ VIN  ───────────────┼──── Servo Red    (5V)
│ GND  ───────────────┼──┬─ Servo Brown  (GND)
│                     │  └─ OLED Blue   (GND)
│ GPIO13 ─────────────┼──── Servo Orange (Signal)
│ 3.3V ───────────────┼──── OLED Red    (VCC)
│ GPIO22 (SCL) ───────┼──── OLED Orange (SCL)
│ GPIO21 (SDA) ───────┼──── OLED White  (SDA)
└─────────────────────┘
```

---

## OLED Display

Shows current servo angle (large) and status (small):

```
┌──────────────────┐
│ Servo            │
│ 180 deg          │
│ TRIGGERED        │
└──────────────────┘
```

Status: `TRIGGERED` when at 180°, `STANDBY` otherwise.

---

## Home Assistant Entities

| Entity | Type | Function |
|--------|------|----------|
| `button.servo_trigger` | Button | Move to 180°, wait 1s, return to 0° |
| `number.servo_angle_set` | Number (0–180) | Set any angle manually |
| `sensor.servo_angle` | Sensor | Current angle in degrees |

---

## Setup

### 1. Install ESPHome

```bash
pip install esphome
```

### 2. Configure secrets

```bash
cp secrets.yaml.template secrets.yaml
# Edit secrets.yaml with your WiFi credentials and API key
```

Generate an API key:
```bash
esphome generate-api-key
```

### 3. Flash (first time via USB)

```bash
esphome run servo-oled.yaml
```

Subsequent updates are OTA:
```bash
esphome upload servo-oled.yaml
```

### 4. Add to Home Assistant

HA auto-discovers the device. Go to:
**Settings → Devices & Services → ESPHome → Add**

Enter the API key from your `secrets.yaml`.

---

## Serial Monitor

```bash
esphome logs servo-oled.yaml
```

Expected output on trigger:
```
[I][servo:0] Moving to 180 deg (level=1.000)
[I][servo:0] Moving to 0 deg (level=-1.000)
```

---

## HA Automation Example

```yaml
- alias: "Servo bei Ereignis auslösen"
  trigger:
    - platform: state
      entity_id: input_boolean.my_trigger
      to: "on"
  action:
    - service: button.press
      target:
        entity_id: button.servo_oled_servo_trigger
```
