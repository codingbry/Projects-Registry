# SongHub ESP32 BLE Remote - Implementation Report

## Project status

The SongHub ESP32 firmware project was added in Git commit `8d8931a` (`Add SongHub ESP32 firmware project`) in the [Songhub-ESP32 source repository](../../../Songhub/Songhub-ESP32/). The implementation is source-complete enough for PlatformIO build and upload preparation, but the project documentation explicitly states that physical keypad and enclosure testing is still pending.

**Status:** Implemented; build and hardware verification pending.

## Scope of the change

The project adds a Bluetooth Low Energy remote for an original ESP32 DevKit / ESP32-WROOM-32. It reads a 4 x 5 keypad matrix and sends keyboard, media-control, and SongHub-specific function-key events to the host computer.

This project is separate from the earlier ESP8266 Wi-Fi/WebSocket remote documented in [`SongHub-ESP8266-Remote.md`](./SongHub-ESP8266-Remote.md). It is also not interchangeable with the ESP32-C3 binary bundle under the broader Songhub folder: the board family, GPIO mapping, and transport layer differ.

## Files added

| File | Purpose |
|---|---|
| [`Songhub-ESP32.ino`](../../../Songhub/Songhub-ESP32/Songhub-ESP32.ino) | Arduino firmware entry point and keypad event handling |
| [`config.h`](../../../Songhub/Songhub-ESP32/config.h) | GPIO mapping, key map, BLE passkey, debounce, and sleep settings |
| [`platformio.ini`](../../../Songhub/Songhub-ESP32/platformio.ini) | PlatformIO board, framework, library, monitor, and upload configuration |
| [`README.md`](../../../Songhub/Songhub-ESP32/README.md) | Build, wiring, upload, pairing, operation, troubleshooting, and Tagalog user guide |
| [`.gitignore`](../../../Songhub/Songhub-ESP32/.gitignore) | Excludes PlatformIO-generated output |

## Hardware and wiring

The firmware targets an original ESP32 DevKit / ESP32-WROOM-32. It must not be flashed to an ESP32-C3, ESP32-S2, ESP32-S3, or ESP8266 without a board-specific firmware change.

The 4 x 5 matrix is configured as follows:

| Matrix wire | GPIO |
|---|---:|
| Row 1 | 13 |
| Row 2 | 14 |
| Row 3 | 16 |
| Row 4 | 17 |
| Column 1 | 18 |
| Column 2 | 19 |
| Column 3 | 21 |
| Column 4 | 22 |
| Column 5 | 23 |

The key layout is:

| Row | Buttons |
|---|---|
| 1 | BGV, 1, 2, 3, Melody |
| 2 | Volume+, 4, 5, 6, Tempo+ |
| 3 | Volume-, 7, 8, 9, Tempo- |
| 4 | Next, Stop, 0, Play/Pause, Reserve |

The project uses 3.3 V ESP32 GPIO logic. GPIO6-GPIO11 are avoided because they are reserved for the module SPI flash.

## Firmware behavior

The firmware advertises as `SongHub ESP32 Remote` using the BLE Keyboard library. It only sends events when a BLE host is connected.

| Button or input | Event sent |
|---|---|
| `0`-`9` | Keyboard number |
| Volume+ / Volume- | Standard media volume up/down |
| Next | Standard next-track media key |
| Play/Pause | Standard play/pause media key |
| BGV | F13 |
| Melody | F14 |
| Tempo+ | F15 |
| Tempo- | F16 |
| Stop | F17 |
| Reserve | F18 |

The host SongHub software must be configured to interpret F13-F18 for the custom actions. The BLE pairing passkey in the committed configuration is `123456`; it should be changed before shared or public deployment.

Deep sleep is deliberately disabled. The source leaves sleep as a follow-up because a keypad matrix requires tested wake lines and debouncing behavior when a key is held.

## Build and upload configuration

PlatformIO is configured for the `esp32dev` environment using the Espressif32 platform and Arduino framework. Required dependencies are:

- `Keypad` version `^3.1.1`
- `ESP32-BLE-Keyboard` from the upstream GitHub repository

The configured serial monitor speed is `115200`, and the default upload speed is `921600`. The project keeps the sketch and headers at the project root and restricts the build source filter to `Songhub-ESP32.ino`.

## Verification state

The Git commit verifies that the source project, configuration, and user documentation were added together. The reviewed project documentation does not claim a completed physical test.

Verification still required:

1. Run a PlatformIO build and confirm dependency/API compatibility.
2. Upload to the exact original ESP32 DevKit/WROOM-32 board.
3. Confirm Bluetooth pairing with the expected passkey.
4. Test every matrix key and confirm row/column wiring.
5. Confirm number, media, and F13-F18 events in the SongHub host software.
6. Test behavior after reset and confirm that no key is held during boot.

## Related Songhub implementations

- The [ESP8266 Wi-Fi remote report](./SongHub-ESP8266-Remote.md) covers the NodeMCU ESP8266 WebSocket/OLED implementation, which remains compile and hardware-test pending.
- The broader [Songhub folder](../../../Songhub/) also contains an ESP32-C3 flash-and-wiring bundle. Its packaged binary and wiring are board-specific and must not be mixed with this original ESP32 BLE source project.

