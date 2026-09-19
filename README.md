# CANBus Triple — PlatformIO port

This is the original [CANBus-Triple/CANBus-Triple](https://github.com/CANBus-Triple/CANBus-Triple)
firmware, repackaged as a PlatformIO project so it builds on a current,
maintained toolchain instead of the frozen Arduino IDE 1.6.1 "hardware/"
package the original repo shipped.

## What changed vs. the original repo

- **Build system**: Arduino IDE 1.6.1 manual hardware-folder install → PlatformIO
  project (`platformio.ini` + `boards/canbustriple.json`). This is the actual
  fix for "abandoned" — the old `platform.txt`/`boards.txt` pair pinned nothing
  reproducibly and relied on whatever AVR core happened to be installed via
  Arduino's Boards Manager.
- **AVR core / SPI / EEPROM / Wire / SoftwareSerial**: previously bundled as
  frozen 2016 copies inside `avr/cores` and `avr/libraries`. These are now
  dropped entirely — PlatformIO's `atmelavr` platform supplies current,
  maintained versions of all of these automatically. I checked the actual
  calls this firmware makes (`SPI.transfer`, `SPI.setDataMode`,
  `SPI.setClockDivider`, `SPI.setBitOrder`, `EEPROM.read`/`write`,
  `avr/wdt.h`, `avr/sleep.h`, `avr/power.h`) — all of it is still present
  and supported in the current AVR core, so **no source changes were needed**
  in the firmware logic itself to make this swap.
- **Custom board + pin mapping**: carried over unchanged into
  `boards/canbustriple.json` (translated from `boards.txt`/`platform.txt`)
  and `variants/canbustriple/pins_arduino.h` (copied byte-for-byte — it's
  pure pin/register mapping, nothing toolchain-version-specific).
- **CANBus (MCP2515) driver library**: copied unchanged into `lib/CANBus/`.
- **Firmware sketch**: `CANBusTriple.ino` and its headers
  (`Middleware.h`, `Settings.h`, `AutoBaud.h`, `SerialCommand.h`,
  `ServiceCall.h`, `ChannelSwap.h`, `Naptime.h`) copied unchanged into `src/`.
- **Bootloader**: only the prebuilt `Caterina.hex` is kept, under
  `extras/bootloader/`, as a fallback in case you ever need to re-flash the
  bootloader itself. You almost certainly won't need this for normal firmware
  development — uploads go through the bootloader that's already on the chip.
  The original LUFA bootloader *source* (16 MB) was left out to keep this
  package small; it's still in the original repo if you ever need to modify
  the bootloader itself.
- **Mazda example**: kept as reference under `examples/CANBusTriple_Mazda/`,
  not built by default.

## ⚠️ Important — this has not been compile-tested

I don't have network access or an AVR toolchain in this environment, so I
could not actually run a build here. I read through the whole firmware and
the SPI/EEPROM/watchdog calls it makes are all still valid against the
current AVR core, and the board/variant JSON was translated field-for-field
from the original `boards.txt`/`platform.txt`, so this should build — but
"should" is doing some work in that sentence. **Please build it locally and
send me the output** (especially any errors) so we can fix anything that
doesn't compile cleanly on the first try. This is a normal part of a port
like this, not a sign something went wrong.

## Building

1. Install [PlatformIO](https://platformio.org/install) (via VS Code
   extension, or `pip install platformio` for the CLI).
2. From this project's root directory:
   ```
   pio run
   ```
3. To flash it to your CANBus Triple (put it in bootloader mode first if it
   doesn't auto-reset — see below):
   ```
   pio run --target upload
   ```

If `pio run --target upload` doesn't find your board automatically, specify
the port:
```
pio run --target upload --upload-port /dev/ttyACM0   # Linux example
pio run --target upload --upload-port COM5           # Windows example
```

### If upload fails to find the bootloader

The board uses the Caterina bootloader (same family as Arduino Leonardo) and
a 1200-baud-touch reset, which PlatformIO should trigger automatically. If it
doesn't, double-tap the physical reset button on the CANBus Triple right
before running the upload command — that forces it into bootloader mode.

## Project layout

```
platformio.ini              Build config
boards/canbustriple.json    Custom board definition (MCU, fuses, upload protocol)
variants/canbustriple/      Custom pin mapping (unchanged from original)
lib/CANBus/                 MCP2515 CAN controller driver (unchanged)
src/                        Main firmware (unchanged)
examples/CANBusTriple_Mazda/  Mazda-specific example, for reference
extras/bootloader/          Prebuilt Caterina bootloader hex, fallback only
```

## Adding features on top

Once you've confirmed this builds and flashes cleanly, this is a normal
PlatformIO project — add new `.cpp`/`.h` files to `src/`, or new libraries
to `lib/`, same as any other PlatformIO Arduino-framework project. Come back
here with what you want to add and we can go from there.
