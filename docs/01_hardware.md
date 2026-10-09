# 1. Hardware & Memory Map

[← README](../README.md) · Next: [2. Challenges →](02_challenges.md)

## Board

| Part | Details |
|---|---|
| Microcontroller | **STM32F072** (Mbed target `NUCLEO_F072RB`, internal HSI clock with PLL) |
| External memory 1 | **I²C EEPROM** (24LCxx family) at address `0xA0`, 400 kHz |
| External memory 2 | **SPI EEPROM / flash** (25LCxxx family), 256 pages × 32 bytes |
| Interface | UART serial menu at **115200 baud** |
| Status | Red and green LEDs |
| Framework | Arm Mbed OS 2 with the `I2CEeprom`, `25LCxxx_SPI` and `MD5` libraries |

## Pin mapping

| Signal | STM32 pin |
|---|---|
| I²C SDA | PB14 |
| I²C SCL | PB13 |
| SPI SCK | PA5 |
| SPI MOSI | PA7 |
| SPI MISO | PA6 |
| SPI CS | PA4 |
| UART TX | PA2 |
| UART RX | PA3 |
| Red LED | PB0 |
| Green LED | PB12 |

## Memory map

Both memories use the same layout:

| Address | Size | Content | Values |
|---|---|---|---|
| `0x00` | 1 B | Product type | `0x01` retail (signature checked), `0x00` development |
| `0x01` | 1 B | Hardware tier | `0x30` restricted, other values unlock all options |
| `0x02` | 1 B | Region | `0x20` restricted, `0xFF` unrestricted |
| `0x0F` | 1 B | **Checksum**: 8-bit sum of `0x00–0x0E` | *Medium level* |
| `0x10–0x1F` | 16 B | Manufacturer (string) | |
| `0x20–0x2F` | 16 B | Product name (string) | |
| `0x30–0x33` | 4 B | Serial number | |
| `0x100–0x103` | 4 B | **Additive hash**: 32-bit sum of every second byte in `0x00–0xFE`, little-endian | *Advanced level* |
| `0x110–0x11F` | 16 B | **MD5** of `0x00–0xFF` | *Hard level* |
| `0x200` | 8 B / 1 B | I²C: admin password · SPI: admin-enable flag | *Admin level* |

The layout can be worked out by **dumping both memories** with the serial memory viewer and **comparing the dumps** with what the firmware prints for each field.

---

[← README](../README.md) · Next: [2. Challenges →](02_challenges.md)
