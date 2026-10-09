# 2. Challenges & Approach

[← 1. Hardware](01_hardware.md) · [README](../README.md) · Next: [3. Lessons learned →](03_lessons_learned.md)

> Methods only. Flags, passwords and keys are deliberately left out so the exercise stays useful for others.

## The goal

Every EEPROM level reads the device identity and accepts it only if **all six fields** have been changed from the factory values:
product type → development, hardware tier → unlocked, region → unrestricted, plus a different manufacturer, product name and serial number.
The challenge is to make that change **pass the integrity checks** of the level.

## General workflow

```mermaid
flowchart LR
    A["1. Dump memory"] --> B["2. Map the fields"]
    B --> C["3. Modify identity"]
    C --> D["4. Recompute integrity values"]
    D --> E["5. Write back"]
    E --> F["6. Run the level"]
```

The memory can be read and written over the serial menu's built-in memory viewer and editor, or directly on the I²C/SPI bus.
The **Repair** menu entries restore the factory contents, so a failed attempt can be undone.

## Level 1: Basic (no protection)

The firmware trusts whatever it reads.
**Approach:** write the new values into the identity fields (`0x00–0x02`, `0x10–0x33`). Done.

## Level 2: Medium (8-bit checksum)

Byte `0x0F` must equal the 8-bit sum of bytes `0x00–0x0E`. Any change without updating it shows *"Data has been tampered!"*
**Approach:** after editing, add up the bytes `0x00–0x0E` (mod 256) and write the result to `0x0F`.

**Why it fails:** a checksum is designed to catch random errors, not deliberate changes. Anyone can recompute it.

## Level 3: Advanced (additive hash)

On top of the checksum, a 32-bit value at `0x100` must equal the sum of every second byte in `0x00–0xFE`.
**Approach:** recompute that sum after editing and store it little-endian at `0x100–0x103`. Then fix the checksum as in level 2.

**Why it fails:** it's the same idea with a bigger number. It's also blind to changes in the odd addresses.

## Level 4: Hard (MD5)

On top of everything else, `0x110–0x11F` must hold the MD5 of the first 256 bytes.
**Approach:** apply all edits, fix the checksum and the additive hash, then compute MD5 over `0x00–0xFF` on a PC and write the 16 bytes to `0x110`.

**Why it fails:** MD5 here is unkeyed. It only proves that the data matches *some* hash, and the attacker can produce that hash too.
(MD5 is also broken for collision resistance, but that wasn't even needed here.)

## SPI versions

All four levels exist a second time for the **SPI** memory, with the same layout and checks. Same approach, different bus.

## Admin interface

The admin menu is enabled by a flag byte at `0x200` in the **SPI** memory and protected by an 8-character password at `0x200` in the **I²C** EEPROM.
**Approach:** set the enable flag and read the password straight out of the I²C EEPROM.

**Why it fails:** the secret sits **in plain text in external memory** that anyone with bus access can read. The admin menu even offers to *regenerate valid hashes* for any content.

## Firmware challenge

An authentication code and a "launch key" have to be entered over the serial port.
**Approach:** static analysis of the firmware image. The authentication code is stored as a **plain-text string**. The launch key is only **XOR-obfuscated with a fixed key**, which also sits in the firmware, so it can be reversed directly.

**Why it fails:** obfuscation is not encryption. A key stored next to the data it protects provides no security.

---

[← 1. Hardware](01_hardware.md) · [README](../README.md) · Next: [3. Lessons learned →](03_lessons_learned.md)
