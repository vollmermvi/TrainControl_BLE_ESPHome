# TrainControl ESPHome

ESPHome-based firmware for Dragon Railway locomotives with Home Assistant integration.

**Forked from [DragonRailway/TrainControl_BLE](https://github.com/DragonRailway/TrainControl_BLE)** - converted from RemoteXY/BLE to ESPHome/WiFi for better reliability and Home Assistant orchestration.

## Table of Contents

- [Why ESPHome?](#why-esphome)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
  - [Step 1: Set Up Home Assistant](#step-1-set-up-home-assistant)
  - [Step 2: Install ESPHome Add-on](#step-2-install-esphome-add-on)
  - [Step 3: Configure the Firmware](#step-3-configure-the-firmware)
  - [Step 4: Flash the TrackLink Board](#step-4-flash-the-tracklink-board)
  - [Step 5: Add Device to Home Assistant](#step-5-add-device-to-home-assistant)
  - [Step 6: Set Up the Dashboard](#step-6-set-up-the-dashboard)
  - [Step 7: Configure Audio (Optional)](#step-7-configure-audio-optional)
- [Usage](#usage)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)
- [Future Expansion](#future-expansion)
- [Credits](#credits)

---

## Why ESPHome?

The original RemoteXY implementation had session disconnect issues (especially on iOS devices going idle). This fork uses ESPHome + Home Assistant for:

- **Reliable connectivity** - WiFi with automatic reconnection
- **Home Assistant integration** - Control from HA dashboards, automations, voice assistants
- **Multi-device orchestration** - Coordinate multiple locomotives, crossings, signals
- **Audio support** - Train sounds via MAX98357 amplifier
- **No app required** - Control from any browser or HA app

---

## Features

- Motor control with variable speed and direction
- 6 LED channels with smooth fade effects
  - Headlight
  - Taillight
  - Cab light
  - Walk light
  - Ditch lights (left/right with alternating flash)
- Battery voltage monitoring with low battery warnings
- I2S audio output (MAX98357) for train sounds
- Emergency stop button
- Home Assistant media player for sound control
- OTA (Over-The-Air) firmware updates

---

## Hardware Requirements

### TrackLink V2 Board (Recommended)

| Component | Details |
|-----------|---------|
| MCU | ESP32-S3 (Lolin S3 Mini) |
| Motor Driver | DRV8212 (4A peak) |
| LEDs | 6 channels with 100Ω resistors |
| Audio | MAX98357 I2S amplifier |
| Battery | 1-3S LiPo with voltage sensing |

### Pin Mapping Reference

| Function | GPIO Pin |
|----------|----------|
| Motor PWM1 | 6 |
| Motor PWM2 | 7 |
| Motor Enable | 5 |
| Headlight (L1) | 9 |
| Taillight (L2) | 10 |
| Cab Light (L3) | 11 |
| Walk Light (L4) | 12 |
| Ditch Light L (L5) | 13 |
| Ditch Light R (L6) | 14 |
| Audio LRCLK | 15 |
| Audio BCLK | 16 |
| Audio DIN | 17 |
| Audio SD_MODE | 18 |
| Battery Sense | 4 |
| Power Enable | 21 |
| Power Button | 38 |
| Charge Sense | 48 |

### Other Requirements

- USB-C cable for initial flashing
- WiFi network (2.4GHz)
- Computer with web browser
- Home Assistant instance (Raspberry Pi, VM, or dedicated server)

---

## Prerequisites

Before starting, ensure you have:

1. **Home Assistant** installed and running
   - Minimum version: 2023.x or later
   - Accessible via web browser

2. **Network access**
   - Home Assistant and the locomotive will be on the same WiFi network
   - Know your WiFi SSID and password

3. **TrackLink V2 board**
   - Properly assembled with battery connected
   - USB-C port accessible for initial flash

---

## Deployment Guide

### Step 1: Set Up Home Assistant

If you already have Home Assistant running, skip to Step 2.

#### New Installation

1. Download Home Assistant for your platform: https://www.home-assistant.io/installation/
2. For Raspberry Pi:
   - Flash the HA image to an SD card using [Balena Etcher](https://etcher.balena.io/)
   - Insert SD card and power on the Pi
   - Wait 10-20 minutes for initial setup
3. Access Home Assistant at `http://homeassistant.local:8123`
4. Create your admin account and complete onboarding

---

### Step 2: Install ESPHome Add-on

1. In Home Assistant, go to **Settings** → **Add-ons**

2. Click **Add-on Store** (bottom right)

3. Search for **ESPHome** and click on it

4. Click **Install** and wait for installation

5. After installation:
   - Toggle **Start on boot** → ON
   - Toggle **Watchdog** → ON
   - Click **Start**

6. Click **Open Web UI** to access ESPHome dashboard

   > **Tip:** Add ESPHome to your sidebar: toggle **Show in sidebar** → ON

---

### Step 3: Configure the Firmware

#### 3.1 Create the Configuration File

1. In the ESPHome dashboard, click **+ New Device**

2. Click **Continue** → Enter name: `dragon-locomotive`

3. Select **ESP32-S3** as the device type

4. Click **Skip** (we'll use our own configuration)

5. Find your new device in the list and click **Edit**

6. **Replace the entire contents** with the configuration from this repository:

   Copy everything from: [`esphome/tracklink-v2.yaml`](esphome/tracklink-v2.yaml)

7. Click **Save**

#### 3.2 Create the Secrets File

The configuration uses secrets to keep your credentials safe.

1. In ESPHome dashboard, click the **Secrets** link (top right)

2. Add the following content:

```yaml
# WiFi Configuration
wifi_ssid: "YourWiFiNetworkName"
wifi_password: "YourWiFiPassword"

# ESPHome API encryption key (generate a random one)
# You can generate one at: https://esphome.io/components/api.html
api_key: "your-32-character-base64-key-here"

# OTA update password
ota_password: "choose-a-secure-password"

# Fallback AP password (used if WiFi fails)
ap_password: "fallback-password"
```

3. **Generate an API key:**
   - Visit https://esphome.io/components/api.html
   - Or run: `openssl rand -base64 32`
   - Paste the result as your `api_key`

4. Click **Save**

#### 3.3 Customize the Configuration (Optional)

Edit the main configuration to customize:

```yaml
substitutions:
  name: dragon-locomotive          # Device name (no spaces)
  friendly_name: "Dragon Locomotive"  # Display name in HA
  min_motor_power: "50"            # Minimum motor PWM %
```

---

### Step 4: Flash the TrackLink Board

#### First-Time Flash (USB Required)

1. Connect the TrackLink V2 to your computer via USB-C

2. In ESPHome dashboard, find your device and click the **three dots** menu

3. Click **Install**

4. Select **Plug into this computer**

5. A browser dialog will appear - select the USB serial port
   - Usually appears as "USB Serial" or "CP2102" or similar

   > **Note:** If no port appears, you may need to install drivers:
   > - [CP210x drivers](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers)
   > - [CH340 drivers](https://www.wch.cn/download/CH341SER_EXE.html)

6. Click **Connect** and wait for the flash to complete (2-5 minutes)

7. Watch the log output - you should see:
   ```
   [INFO] Connecting...
   [INFO] Chip ID: ...
   [INFO] Uploading...
   [INFO] Done!
   ```

#### Subsequent Updates (OTA)

After the first flash, updates happen wirelessly:

1. Click **Install** on your device
2. Select **Wirelessly**
3. ESPHome will find the device and update it over WiFi

---

### Step 5: Add Device to Home Assistant

#### Automatic Discovery

1. Go to **Settings** → **Devices & Services**

2. You should see a notification: **"1 new device discovered"**
   - If not, click **+ Add Integration** → search "ESPHome"

3. Click **Configure** on the discovered ESPHome device

4. Enter the API encryption key (same as in secrets.yaml)

5. Click **Submit**

6. The device is now added! Click on it to see all entities.

#### Manual Addition (if auto-discovery fails)

1. Go to **Settings** → **Devices & Services**
2. Click **+ Add Integration**
3. Search for **ESPHome**
4. Enter the device IP address (check your router or ESPHome logs)
5. Enter the API key and submit

#### Verify Entities

After adding, you should see these entities:

| Entity | Type | Description |
|--------|------|-------------|
| `fan.dragon_locomotive_motor` | Fan | Motor speed/direction |
| `light.dragon_locomotive_headlight` | Light | Front light |
| `light.dragon_locomotive_taillight` | Light | Rear light |
| `light.dragon_locomotive_cab_light` | Light | Interior cab |
| `light.dragon_locomotive_walk_light` | Light | Walkway |
| `light.dragon_locomotive_ditch_light_left` | Light | Left ditch |
| `light.dragon_locomotive_ditch_light_right` | Light | Right ditch |
| `sensor.dragon_locomotive_battery_voltage` | Sensor | Battery V |
| `sensor.dragon_locomotive_battery_level` | Sensor | Battery % |
| `button.dragon_locomotive_emergency_stop` | Button | E-stop |
| `media_player.dragon_locomotive_train_speaker` | Media | Audio |

---

### Step 6: Set Up the Dashboard

#### Option A: Quick Setup (Entities Card)

1. Go to your Home Assistant dashboard
2. Click the **three dots** → **Edit Dashboard**
3. Click **+ Add Card**
4. Choose **Entities**
5. Add the locomotive entities
6. Click **Save**

#### Option B: Advanced Dashboard (Recommended)

Use the pre-built dashboard card from this repository:

1. Go to **Settings** → **Dashboards**

2. Create a new dashboard or edit an existing one

3. Click **Edit Dashboard** → **three dots** → **Raw configuration editor**

4. Add the following card configuration (or copy from [`ha-dashboard/train-control-card.yaml`](ha-dashboard/train-control-card.yaml)):

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: "# Dragon Railway"

  - type: horizontal-stack
    cards:
      - type: button
        entity: fan.dragon_locomotive_motor
        name: Motor
        show_state: true
        tap_action:
          action: toggle

  - type: entities
    title: Lights
    show_header_toggle: true
    entities:
      - entity: light.dragon_locomotive_headlight
        name: Headlight
      - entity: light.dragon_locomotive_taillight
        name: Taillight
      - entity: light.dragon_locomotive_cab_light
        name: Cab Light
      - entity: light.dragon_locomotive_walk_light
        name: Walk Light
      - entity: light.dragon_locomotive_ditch_light_left
        name: Ditch Left
      - entity: light.dragon_locomotive_ditch_light_right
        name: Ditch Right

  - type: button
    entity: button.dragon_locomotive_emergency_stop
    name: EMERGENCY STOP
    icon: mdi:alert-octagon
    tap_action:
      action: call-service
      service: button.press
      target:
        entity_id: button.dragon_locomotive_emergency_stop

  - type: entities
    title: Status
    entities:
      - entity: sensor.dragon_locomotive_battery_voltage
      - entity: sensor.dragon_locomotive_battery_level
```

5. Click **Save**

---

### Step 7: Configure Audio (Optional)

The TrackLink V2 has a MAX98357 I2S amplifier for playing train sounds.

#### 7.1 Add Sound Files to Home Assistant

1. Connect to your Home Assistant via SSH or Samba

2. Navigate to the `/media` folder (create it if it doesn't exist):
   ```bash
   mkdir -p /config/media/train-sounds
   ```

3. Upload your train sound files:
   - Supported formats: `.wav`, `.mp3`
   - Recommended: 16-bit, 44.1kHz mono WAV files
   - Example files to add:
     - `whistle.wav`
     - `bell.wav`
     - `horn.wav`
     - `engine-idle.wav`
     - `chug.wav`

#### 7.2 Test Audio Playback

1. Go to **Developer Tools** → **Services**

2. Select service: `media_player.play_media`

3. Configure:
   ```yaml
   service: media_player.play_media
   target:
     entity_id: media_player.dragon_locomotive_train_speaker
   data:
     media_content_id: media-source://media_source/local/train-sounds/whistle.wav
     media_content_type: audio/wav
   ```

4. Click **Call Service**

5. You should hear the sound from the locomotive!

#### 7.3 Create Audio Buttons on Dashboard

Add buttons to your dashboard for easy sound control:

```yaml
type: horizontal-stack
cards:
  - type: button
    name: Whistle
    icon: mdi:whistle
    tap_action:
      action: call-service
      service: media_player.play_media
      target:
        entity_id: media_player.dragon_locomotive_train_speaker
      data:
        media_content_id: media-source://media_source/local/train-sounds/whistle.wav
        media_content_type: audio/wav

  - type: button
    name: Bell
    icon: mdi:bell
    tap_action:
      action: call-service
      service: media_player.play_media
      target:
        entity_id: media_player.dragon_locomotive_train_speaker
      data:
        media_content_id: media-source://media_source/local/train-sounds/bell.wav
        media_content_type: audio/wav

  - type: button
    name: Horn
    icon: mdi:bullhorn
    tap_action:
      action: call-service
      service: media_player.play_media
      target:
        entity_id: media_player.dragon_locomotive_train_speaker
      data:
        media_content_id: media-source://media_source/local/train-sounds/horn.wav
        media_content_type: audio/wav
```

#### 7.4 Create Automations with Sound

Example: Play whistle when motor starts

1. Go to **Settings** → **Automations & Scenes**

2. Click **+ Create Automation**

3. Configure:
   - **Trigger:** State change - `fan.dragon_locomotive_motor` from `off` to `on`
   - **Action:** Call service `media_player.play_media` with whistle sound

```yaml
alias: Train Whistle on Start
trigger:
  - platform: state
    entity_id: fan.dragon_locomotive_motor
    from: "off"
    to: "on"
action:
  - service: media_player.play_media
    target:
      entity_id: media_player.dragon_locomotive_train_speaker
    data:
      media_content_id: media-source://media_source/local/train-sounds/whistle.wav
      media_content_type: audio/wav
```

---

## Usage

### Basic Controls

| Action | How To |
|--------|--------|
| Start motor | Toggle motor fan entity ON, adjust speed slider |
| Change direction | Use the direction control on the fan entity |
| Turn on lights | Toggle individual light entities or use "All Lights" button |
| Emergency stop | Press the Emergency Stop button |
| Play sound | Use media player or sound buttons |

### Motor Speed Control

The motor uses a fan entity with speed control:
- Speed range: 0-100%
- Minimum effective speed is set by `min_motor_power` (default 50%)
- Below minimum, motor won't move but above it scales linearly

### Battery Monitoring

- **Voltage sensor:** Shows actual battery voltage
- **Level sensor:** Shows estimated percentage (based on 1S LiPo: 3.0V=0%, 4.2V=100%)
- Low battery warning appears in logs when voltage drops

---

## Customization

### Change Device Name

Edit the substitutions in `tracklink-v2.yaml`:

```yaml
substitutions:
  name: my-train-name
  friendly_name: "My Train Name"
```

### Adjust LED Brightness

Modify the light transition or add brightness limits:

```yaml
light:
  - platform: monochromatic
    name: "Headlight"
    output: led_headlight
    default_transition_length: 1s  # Slower fade
    gamma_correct: 2.0  # Adjust brightness curve
```

### Add Multiple Locomotives

1. Duplicate the configuration file
2. Change the `name` substitution to be unique
3. Flash each TrackLink board separately
4. Each will appear as a separate device in Home Assistant

---

## Troubleshooting

### Device won't connect to WiFi

1. Verify WiFi credentials in `secrets.yaml`
2. Ensure 2.4GHz network (ESP32 doesn't support 5GHz)
3. Check router isn't blocking new devices
4. Connect via USB and check logs in ESPHome

### Device not discovered in Home Assistant

1. Ensure device and HA are on same network
2. Check if mDNS is working (try pinging `dragon-locomotive.local`)
3. Manually add by IP address
4. Verify API key matches between device and HA

### Motor doesn't respond

1. Check battery is charged and connected
2. Verify motor connections to TrackLink board
3. Check logs for errors: ESPHome dashboard → Logs
4. Try emergency stop then re-enable

### Audio not working

1. Verify speaker is connected to MAX98357 output
2. Check `audio_enable` output is turning on
3. Try different audio file formats
4. Check volume isn't muted in media player

### OTA updates fail

1. Ensure device is powered and connected to WiFi
2. Check available flash memory
3. Try power cycling the device
4. Fall back to USB flash if needed

### Viewing Logs

1. ESPHome dashboard → Click **Logs** on your device
2. Or in Home Assistant: **Developer Tools** → **Logs** → filter by device name

---

## Future Expansion

This setup is designed for a complete model railway ecosystem:

### Additional Devices

| Device | Purpose | Components |
|--------|---------|------------|
| Crossing | Flashing lights + gate | ESP32 + LEDs + Servo |
| Signal | Track signals | ESP32 + RGB LEDs |
| Station | Lighting + sounds | ESP32 + LEDs + Speaker |
| Track Sensor | Train detection | ESP32 + IR sensors |

### Example Automation Ideas

- **Automatic crossing:** When train approaches, lower gates and flash lights
- **Station announcement:** Play sounds when train arrives at station
- **Night mode:** Dim all lights at scheduled times
- **Demo mode:** Run choreographed sequence for display

All devices can be coordinated through Home Assistant automations.

---

## Project Structure

```
TrainControl_BLE_ESPHome/
├── esphome/
│   ├── tracklink-v2.yaml      # Main ESPHome configuration
│   ├── secrets.yaml.example   # Template for credentials
│   ├── .gitignore             # Ignores secrets.yaml
│   └── sounds/                # Train sound files (optional)
├── ha-dashboard/
│   └── train-control-card.yaml # Home Assistant dashboard card
├── locomotive_configs/         # Original RemoteXY configs (reference)
├── photos/                     # Screenshots
├── platformio.ini              # Original PlatformIO config (reference)
└── README.md                   # This file
```

---

## Credits

- **Original firmware:** [DragonRailway](https://github.com/DragonRailway)
- **Hardware design:** [RamBros 3D](https://blog.rambros3d.com/)
- **ESPHome conversion:** This fork

---

## License

MIT License - See [LICENSE](LICENSE)

---

## Support

- **Issues:** Open an issue on this GitHub repository
- **Dragon Railway community:** [DragonRailway GitHub](https://github.com/DragonRailway)
- **ESPHome documentation:** https://esphome.io/
- **Home Assistant documentation:** https://www.home-assistant.io/docs/
