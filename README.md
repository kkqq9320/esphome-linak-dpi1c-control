<!-- Control the Liank DPI1C via ESPHome in Home Assistant. -->
# ESPhome LINAK DPI1C Control

[DPI1C](https://www.linak-us.com/segments/deskline/start/dpi1c/)

Bluetooth Low Energy (BLE) integration for LINAK DPI1C control using ESPHome.
This package allows you to control your standing desk, save/restore presets, adjust the key lock settings, and monitor the desk's height and motor status directly from Home Assistant via ESPHome.

**Key Characteristic:** This integration controls the desk in the exact same manner as the official app, allowing for highly precise positioning (down to 0.1cm accuracy).

> [!CAUTION]
> **USE AT YOUR OWN RISK**
> 
> By using this custom integration, you acknowledge and accept all associated risks. The author is not responsible for any direct or indirect physical damage, hardware malfunctions, or voided warranties that may occur to your LINAK equipment.
>
> **This is an unofficial, personal project and is in no way affiliated with, endorsed by, or related to LINAK.**

> [!WARNING]
> **Single Desk per ESP32 Recommended**
> 
> Connecting multiple desks to a single ESP32 board is untested and discouraged. Due to the high-frequency BLE traffic required for precision control and ESP32 hardware limitations, connection stability cannot be guaranteed in multi-desk configurations. It is strongly advised to dedicate one ESP32 board per desk.

> [!IMPORTANT]
> **Prerequisites & Limitations**
> 
> While this package supports most controller features natively, **you MUST use the official `Desk Connect` app (NOT `Desk Control`) for the initial setup**. You need to use the app first to define the maximum/minimum heights and any height limit boundaries. 

## Features
- **High Precision Control & Monitoring**: Real-time height and speed tracking. Moves to exact target heights (0.1cm granularity), identical to the official app's behavior.
- **Preset Management**: Load, save, and delete up to 4 presets (syncs with the controller).
- **Physical Stop Detection**: Cancels the moving process if physical button intervention or collision is detected (Note: An E16 error will briefly appear on the controller's display, but it recovers automatically. The official app exhibits the exact same E16 error behavior when physically interrupted during movement).
- **Key Lock Settings**: Manage auto-lock timeout and unlock combinations directly from Home Assistant (Automatically restores settings currently applied to the device upon pairing).
- **Desk Name Display**: Reads and displays the desk's physical name from the controller.
- **Interval Heartbeat**: Maintains connection.

### Unsupported Features (By Design)
- **One-Touch Operation & Desk Renaming**: Setting the "one-touch" drive setting or changing the desk's internal name via Home Assistant is not supported, simply because I did not personally need these features.
- **Reminders**: The standing reminder feature is not implemented. While it could theoretically be added, it is much better and more flexible to handle reminders using Home Assistant's native automations.

## Available Entities

Once loaded into Home Assistant, the following entities will be available:

**Sensors:** Height Sensor, Speed Sensor, Preset Heights 1-4, Desk Name, Motor Direction, Moving Status.
**Controls:** 
  - **Number** : Input (Target Height)
  - **Buttons** : Move (Up/Down/Stop), Preset Buttons (Load/Save/Delete)
  - **Switches** : BLE Connection, Key Lock (Enable/Disable)
  - **Selects** : Key Lock Timeout Length, Key Lock Unlock Mode


## Requirements
- ESP32 board with BLE capabilities
- A LINAK desk controller using DPI1C
- ESPHome installed (v2023.12.0 or higher recommended)
- Your desk's MAC address (This can be found via the `Desk Connect` app on your smartphone, or through a BLE scanner app).

## Installation

Add the following to your ESPHome device configuration `yaml` file. 

> [!NOTE]
> This package only contains the desk logic. **You must still configure your board's basic settings** (such as `wifi`, `api`, `ota`, and your specific `esp32` board type) according to your own environment.
> *(Note: `esp32_ble_tracker` with its `scan_parameters: active: true` setting is already included within this package, so you do **not** need to add it separately to your main file.)*

```yaml
packages:
  esphome_linak_dpi1c:
    url: https://github.com/kkqq9320/esphome-linak-dpi1c-control/
    files:
      - path: esphome_linak_dpi1c.yaml
        vars:
          dpi1c_desk: "Desker"                                      # Your desk's friendly name (Device UI Name)
          desk_id: "desker_id1"                                     # Unique ID of the desk (used internally in YAML). Strictly lowercase, use underscores instead of spaces.
          mac_addr: "CA:07:XX:XX:XX:XX"                             # Your desk's MAC Address
          
          # --- Optional: Logger Settings ---
          # `INFO` (Default): Shows basic status, connection, and preset save/load logs. Good for daily use.
          # `DEBUG`: Shows highly detailed logs including real-time height updates during movement and BLE handshake processes. Useful for troubleshooting.
          log_level: "INFO" 

      # Connecting two desks to a single board might cause stability issues.
      # This configuration is **UNTESTED**.
      # Due to heavy BLE traffic and the hardware limitations of the ESP board, 
      # connection drops are highly likely. 
      #   It is strongly recommended to use 1 board per desk.
      # - path: esphome_linak_dpi1c.yaml                            # Load the same file again for the second desk
      #   vars:
      #     dpi1c_desk: "XXXX"
      #     desk_id: "xxxx"
      #     mac_addr: "XX:XX:XX:XX:XX:XX"

```

## How to Connect & Setup

Follow these exact steps to pair the ESP32 board to your desk for the first time:

1. Flash your ESP32 board with your ESPHome configuration (either by using `packages:` as described above, or copying the source code directly).
2. Wait for the ESP board to boot up. Watch the ESPHome logs until you see that it is actively scanning for BLE devices.
3. Once scanning begins, press and hold the physical button on the back/bottom of your DPI1C desk controller for 2~3 seconds until it enters pairing mode and displays its ID (e.g., `ID: XXXX`) on the screen.
4. The ESP board will automatically handshake and pair. You should immediately see the Height values start streaming in the logs and changing in Home Assistant.

## Special Behaviors & Troubleshooting

### Pairing Issues ("Connected successfully" but no real connection)
If your ESPHome logs show `Connected successfully` but the data stream / pairing process doesn't actually begin:
1. Turn **OFF** the BLE Connection switch in Home Assistant / ESPHome.
2. Turn the BLE Connection switch back **ON**.
3. Press and hold your desk controller's physical pairing button until the Bluetooth ID and Name appear on the display (forcing it into pairing mode). The ESP should now properly handshake.

### Using the Mobile App while ESP is active
If you ever need to connect the official `Desk Connect` mobile app while the ESP is already running:
1. Turn **OFF** the BLE Connection switch via Home Assistant.
2. Connect your mobile app as usual.
3. Once finished, temporarily turn **OFF** your phone's Bluetooth to fully disconnect it.
4. Turn the BLE Connection switch **ON** in Home Assistant.
5. Enter the desk's physical pairing mode again (by pressing and holding the button) to allow the ESP32 to reconnect successfully.

### Understanding Presets
When switching between this ESP integration and the official app, preset behaviors can seem confusing. The key rule to remember is: **The desk's internal memory always takes priority.** 
For example, if you register a preset via the ESP board, disconnect the ESP, pair the mobile app to set a different preset, and then reconnect the ESP to set yet another distinct preset... when parsing the presets back on the mobile app, it may temporarily display outdated information. Ultimately, the true active preset is what the physical desk remembers.




## Project Rationale & Integrations

> [!NOTE]
> **Why ESPHome and not a Custom Integration?**
> While the packets might look similar to existing [official LINAK integrations](https://www.home-assistant.io/integrations/linak/), the underlying operating mechanism for the DPI1C series is significantly different. 
Due to these protocol variations, it is unlikely this project can be seamlessly merged with existing Home Assistant integrations. 
I currently have no plans to convert this ESPHome package into a Home Assistant Custom integration, as the present implementation provides the desired precision and control.

## Credits

Enjoy controlling your LINAK desk!
