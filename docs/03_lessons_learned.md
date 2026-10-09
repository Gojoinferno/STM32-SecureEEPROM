# 3. Lessons Learned

[← 2. Challenges](02_challenges.md) · [README](../README.md)

## What went wrong

| Weakness on the board | Why it's a problem |
|---|---|
| Checksum, additive hash and MD5 with **no secret key** | An attacker who can write the data can also recompute every check |
| **Passwords in plain text** in external EEPROM | Anyone with a probe or a dump can read them |
| Admin mode controlled by a **single flag byte** in external memory | One write unlocks it |
| **Secrets inside the firmware** (plain text or XOR with a fixed key) | Static analysis recovers them |
| A built-in tool that **regenerates valid hashes** | Turns the integrity checks into a formality |
| Security decisions based on **external, unauthenticated data** | The MCU can't tell legitimate data from forged data |

## How to do it properly

1. **Authenticate, don't just check:** protect stored data with a **keyed MAC (HMAC-SHA256 or AES-CMAC)** or an **ECDSA/Ed25519 signature**. Without the key, an attacker can't produce a valid tag.
2. **Keep keys off the bus:** store keys in **protected internal flash** or a **secure element** (for example ATECC608 or an OPTIGA Trust chip). Never in the external EEPROM.
3. **Lock the MCU:** enable **readout protection (RDP)** and disable debug access in production, so the firmware and its secrets can't simply be dumped.
4. **Encrypt confidential data** (AES-GCM gives encryption and authentication together) instead of storing it in plain text.
5. **Never store passwords; store salted hashes** (PBKDF2 or Argon2), and limit login attempts.
6. **Use modern hashes:** SHA-256 instead of MD5, and never a plain sum.
7. **No debug or repair backdoors** in production firmware.

## In one sentence

> Integrity checks catch **accidents**. Only **keys the attacker doesn't have** stop **attackers**.

---

[← 2. Challenges](02_challenges.md) · [README](../README.md)
