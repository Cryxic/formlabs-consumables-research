# Formlabs Consumables Research

Open research into the identification, state tracking, repair, and emulation of **Formlabs resin cartridges** and, eventually, **resin tanks/vats**.

The current focus is the DS2431-based cartridge system used by Form 2 / Form 3-generation printers. The goal is to document how these consumables work and build small, reproducible tools for interoperability, repair, reuse, and preservation.

> This is an independent community research project and is not affiliated with Formlabs.

## Current progress

A Form 3B can successfully read an **Arduino Uno + OneWireHub** as an emulated Formlabs cartridge. By emulating both the DS2431 ROM ID and EEPROM contents, the printer can be made to recognize different cartridge profiles.

| Profile | DS2431 ROM | Observed on Form 3B |
|---|---|---|
| Castable `FLCABL02` | `2D B7 07 BF 23 00 00 7F` | Recognized as Castable |
| White V1 | `2D 43 EA C2 0E 00 00 56` | Recognized as White V1; runtime area set to `FF` reports `0 ml` |
| White V2 `FLGPWH02` | `2D D2 EB C2 0E 00 00 1D` | Recognized as White V2; runtime area set to `FF` reports fresh / `<1000 ml` |

The emulator can also switch between known profiles from the Arduino serial console.

## What we know so far

- Formlabs cartridges tested so far use a Maxim/Analog Devices **DS2431 1-Wire EEPROM**.
- The cartridge exposes two electrical contacts: **GND** and **1-Wire data**.
- A valid cartridge profile requires a matching **ROM ID + EEPROM image**.
- Changing only the DS2431 ROM ID causes otherwise valid cartridge data to stop being recognized.
- Cartridge type information is stored in the early EEPROM region.
- The area around `0x40` is associated with cartridge runtime / consumption state, but its meaning is not identical across resin generations.
- Setting the runtime area to `0xFF` does **not** universally mean "full" or "unused". White V1 and White V2 behave differently, which is why this repo avoids the ambiguous term **blank usage**.
- Commercial "universal cartridge" emulators appear to store transformed profile data rather than a raw DS2431 EEPROM image. A confirmed White V4 emulator dataset has been captured for future analysis.

## Arduino cartridge emulator

Current test hardware:

- Arduino Uno
- [`orgua/OneWireHub`](https://github.com/orgua/OneWireHub)
- Form 3B
- Arduino `D2` connected to cartridge **1-Wire data**
- Arduino `GND` connected to cartridge **GND**

When connected to the printer, the printer provides the 1-Wire pull-up, so the emulator setup does not use an additional external pull-up.

## DS2431 memory notes

The DS2431 exposes 144 readable bytes in the captures used here:

```text
0x0000 - 0x007F   128 bytes user EEPROM
0x0080 - 0x0084   protection / configuration bytes
0x0085            factory byte (observed as 0x55)
0x0086 - 0x008F   additional / reserved bytes
```

Do not write the protection/configuration area on a physical cartridge unless you understand the consequences. Some DS2431 protection settings are permanent.

## Wanted: cartridge dumps

More genuine cartridges are the biggest help right now, especially different resin revisions.

Useful submissions include:

1. full 8-byte DS2431 ROM ID;
2. full `0x0000-0x008F` memory dump;
3. resin name / cartridge code;
4. printer model and firmware;
5. displayed remaining or consumed resin, if known.

Old, empty, expired, or otherwise unusable cartridges are still useful for research.

## Future work

- collect more matched cartridge ROM + EEPROM profiles;
- understand the cartridge consumption/state encoding;
- decode the commercial universal-emulator profile format;
- add more resin types to the Arduino emulator;
- document cartridge hardware and repair;
- extend the same approach to **Formlabs resin tanks/vats** when hardware becomes available.

## Project status

This is active reverse-engineering work. Treat interpretations as provisional unless they are backed by repeatable printer tests.

## License / use

The project is intended for interoperability, repair, preservation, and research on hardware you own. Use experimental firmware and modified consumable profiles at your own risk.
