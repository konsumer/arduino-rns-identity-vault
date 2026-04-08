# arduino-rns-identity-vault

Full working example combining [arduino-rns-password](https://github.com/konsumer/arduino-rns-password) and [arduino-rns-encrypted-store](https://github.com/konsumer/arduino-rns-encrypted-store) on an ESP32 with an SD card.

## What it does

**First boot** (no `/identity.bin` on SD):
- Generates a new Reticulum identity
- Prompts for a password over serial
- Saves the identity encrypted with that password to `/identity.bin`

**Subsequent boots:**
- Prompts for the password
- Restores the identity from SD

**After loading**, three self-tests run automatically:
1. **Announce** — create an announce packet and validate the Ed25519 signature
2. **Packet** — encrypt a packet to the destination, decrypt it, verify payload
3. **Storage** — write a message encrypted to SD, read it back, verify it

Press the board button or send `r` over serial to re-run the tests.

## Hardware

Requires an SD card. Pin assignments are set per-board via `build_flags` in `platformio.ini`. Override any of these defines for your wiring:

| Define | Default (cardputeradv) | Purpose |
|---|---|---|
| `SD_CS_PIN` | 12 | SD chip select |
| `BUTTON_PIN` | 0 | Re-run tests button |
| `SPI_SCK` | 40 | SD SPI clock (optional) |
| `SPI_MISO` | 39 | SD SPI MISO (optional) |
| `SPI_MOSI` | 14 | SD SPI MOSI (optional) |
| `LORA_CS` | 5 | LoRa CS — held HIGH during SD init |
| `SD_SPEED` | 40000000 | SD SPI speed in Hz |

If `SPI_SCK`/`SPI_MISO`/`SPI_MOSI` are not defined, `SD.begin(SD_CS_PIN)` is used with the default SPI bus.

## Supported boards

| PlatformIO env | Board |
|---|---|
| `ttgo-t-beam` | TTGO T-Beam |
| `lilygo_tbeam_supreme` | LilyGo T-Beam Supreme |
| `ttgo-lora32-v21` | TTGO LoRa32 v2.1 |
| `cardputeradv` | M5Stack CardPuter (tested) |

## Build & flash

```sh
pio run -e cardputeradv -t upload
pio device monitor
```

## Offline testing

`identity_tool.py` decrypts identity and message files on a PC using the same algorithms as the device — useful for verifying files without hardware.

```sh
pip install cryptography

# Inspect identity only
python identity_tool.py test/identity.bin

# Inspect identity + decrypt message files
python identity_tool.py test/identity.bin test/msgs/last.enc

# Also print raw key bytes
python identity_tool.py test/identity.bin --show-keys
```

The password for the included test files in `test/` is `1234`.

## Dependencies

- [konsumer/arduino-rns-password](https://github.com/konsumer/arduino-rns-password)
- [konsumer/arduino-rns-encrypted-store](https://github.com/konsumer/arduino-rns-encrypted-store)
- [attermann/microReticulum](https://github.com/attermann/microReticulum)
- [attermann/microStore](https://github.com/attermann/microStore)
- [attermann/Crypto](https://github.com/attermann/Crypto)
