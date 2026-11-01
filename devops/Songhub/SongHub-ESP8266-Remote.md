# SongHub ESP8266 Wi-Fi Remote

## Project overview

This project adapts the SongHub keypad remote for a NodeMCU ESP8266. It sends playback and adjustment commands over the local network through SongHub's WebSocket service, replacing the original Bluetooth connection. The design uses a 4×5 keypad matrix and an I²C SH1106 OLED, with no GPIO expander.

**Status:** Firmware packaged; compile and hardware verification still pending.

## Hardware and firmware

- NodeMCU ESP8266 development board
- 4×5 keypad matrix, based on the ESP32-C3 wired remote layout
- SH1106 128×64 I²C OLED
- Keypad and display are connected directly to the ESP8266. The pin assignments are provisional and use boot-sensitive and UART pins, so they must be checked against the actual board and wiring before assembly.
- Arduino sketch: [`SongHubESP8266.ino`](../../../Songhub/ESP8266/SongHubESP8266/SongHubESP8266.ino)
- End-user instructions: [`ReadMe.md`](../../../Songhub/ESP8266/ReadMe.md)
- Shareable package: [`SongHub-ESP8266-Remote.zip`](../../../Songhub/ESP8266/SongHub-ESP8266-Remote.zip)

The sketch uses the ESP8266 Arduino core and the Keypad, WebSockets, and U8g2 libraries. Its configuration header is stored beside the sketch.

## Network setup and connection

On first startup, or when Wi-Fi credentials are unavailable, the remote starts a setup access point named `SongHub-Setup` with password `songhub123`. Connect a phone to it and open `http://192.168.4.1` to enter the Wi-Fi SSID and password. The remote saves these settings in EEPROM and restarts.

The SongHub host IP is intentionally left blank until entered on the device. Hold **Reserve** to open the OLED/keypad editor. Enter the IP with number keys and use **BGV** for dots; **Volume−** deletes, **Next** switches between IP and port, **Play/Pause** clears the active field, **Reserve** saves, and **Stop** cancels. The WebSocket port defaults to `48085` and can be edited. The display reports Wi-Fi and SongHub connection state, the configured target, and the current song code.

The phone-based setup page only collects Wi-Fi credentials. SongHub's browser remote UI uses port `48081`, but this keypad connects to the SongHub WebSocket service, which was confirmed in the session log to work on port `48085`. The log's development host address, `192.168.100.35`, is not shipped as a default; each user must enter their own host IP.

## Keypad behavior

Number keys buffer a song code. A short **Reserve** press sends that code with the `reserve` action. Other mapped keys send play/pause, next, stop, melody, BGV, volume, and tempo commands. Holding **Reserve** opens the SongHub IP/port editor; holding **BGV** starts Wi-Fi provisioning.

Commands use JSON WebSocket envelopes with an `action` and `payload`. On connection, the firmware requests the database and current state. Song actions send the buffered song-code string; volume and tempo adjustments send numeric payloads.

## Verification and next steps

No compile, upload, or hardware test has been completed. An Arduino/ESP8266 toolchain was unavailable in the documented workspace. Before wider use, compile with the ESP8266 Arduino core, confirm the exact NodeMCU and OLED models and matrix wiring, inspect boot behavior with the selected pins, and test Wi-Fi provisioning and each SongHub command on the local network.

The keypad mapping follows the 4×5 ESP32-C3 remote layout. Dedicated Play Now, Priority, MPX, and BGV controls are not all available as separate keys; revise the mapping if those actions are needed.
