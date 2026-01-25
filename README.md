# TrainControl ESPHome

ESPHome-based firmware for Dragon Railway locomotives with Home Assistant integration.

**Forked from [DragonRailway/TrainControl_BLE](https://github.com/DragonRailway/TrainControl_BLE)** - converted from RemoteXY/BLE to ESPHome/WiFi for better reliability and Home Assistant orchestration.

## Why ESPHome?

The original RemoteXY implementation had session disconnect issues (especially on iOS devices going idle). This fork uses ESPHome + Home Assistant for:

- **Reliable connectivity** - WiFi with automatic reconnection
- **Home Assistant integration** - Control from HA dashboards, automations, voice assistants
- **Multi-device orchestration** - Coordinate multiple locomotives, crossings, signals
- **Audio support** - Train sounds via MAX98357 amplifier
- **No app required** - Control from any browser or HA app

## Features

- Motor control with speed and direction
- 6 LED channels with fade effects (headlight, taillight, cab, walk, ditch lights)
- Alternating ditch light effect
- Battery voltage monitoring with low battery warnings
- I2S audio output (MAX98357) for train sounds
- Emergency stop button
- Home Assistant media player for sound control

## Hardware

### TrackLink V2 (recommended)

This firmware is configured for the **TrackLink V2** board:

| Component | Details |
|-----------|---------|
| MCU | ESP32-S3 (Lolin S3 Mini) |
| Motor Driver | DRV8212 (4A peak) |
| LEDs | 6 channels with 100Ω resistors |
| Audio | MAX98357 I2S amplifier |
| Battery | 1-3S LiPo with voltage sensing |

### Pin Mapping (TrackLink V2)

| Function | GPIO |
|----------|------|
| Motor PWM1 | 6 |
| Motor PWM2 | 7 |
| Motor Enable | 5 |
| LED 1-6 | 9, 10, 11, 12, 13, 14 |
| Audio LRCLK | 15 |
| Audio BCLK | 16 |
| Audio DIN | 17 |
| Audio SD_MODE | 18 |
| Battery Sense | 4 |
| Power Enable | 21 |

## Installation

### Prerequisites

- Home Assistant with ESPHome add-on installed
- TrackLink V2 board (or compatible ESP32-S3)
- WiFi network

### Setup

1. **Copy ESPHome config to your HA instance:**
   ```bash
   cp esphome/tracklink-v2.yaml /config/esphome/
   ```

2. **Create secrets file:**
   ```bash
   cp esphome/secrets.yaml.example /config/esphome/secrets.yaml
   # Edit secrets.yaml with your WiFi credentials and API keys
   ```

3. **Flash the firmware:**
   - Open ESPHome dashboard in Home Assistant
   - Click the three dots on your device → Install
   - Choose "Plug into this computer" for first flash (USB)
   - Subsequent updates can be done OTA

4. **Add to Home Assistant:**
   - The device should auto-discover
   - Or manually add via Integrations → ESPHome

### Dashboard Setup

Import the dashboard card from `ha-dashboard/train-control-card.yaml` into your Lovelace configuration.

## Audio Setup

To play train sounds:

1. Place `.wav` or `.mp3` files in your Home Assistant `/media/` folder
2. Use the media player entity to play sounds:
   ```yaml
   service: media_player.play_media
   target:
     entity_id: media_player.dragon_locomotive_train_speaker
   data:
     media_content_id: media-source://media_source/local/train-whistle.wav
     media_content_type: audio/wav
   ```

3. Or create automations that play sounds on events (horn button, movement start, etc.)

## Future Expansion

This setup is designed for a complete model railway ecosystem:

- **Multiple locomotives** - Each gets its own ESPHome device
- **Crossings** - ESP32 with LEDs and servo for gates
- **Signals** - ESP32 controlling signal lights
- **Track sensors** - Detect train position for automations

All devices coordinate through Home Assistant automations.

## Project Structure

```
├── esphome/
│   ├── tracklink-v2.yaml      # Main ESPHome configuration
│   ├── secrets.yaml.example   # Template for credentials
│   └── sounds/                # Train sound files
├── ha-dashboard/
│   └── train-control-card.yaml # HA Lovelace dashboard
├── locomotive_configs/         # Original RemoteXY configs (reference)
└── README.md
```

## Credits

- Original firmware: [DragonRailway](https://github.com/DragonRailway)
- Hardware design: [RamBros 3D](https://blog.rambros3d.com/)
- ESPHome conversion: This fork

## License

MIT License - See [LICENSE](LICENSE)
