# Parameter packing deep-dive

> This is the nerd corner: how microGranny squeezes eleven parameters × six sounds × multiple presets into EEPROM-friendly bitfields.

## Layout constants

* `NUMBER_OF_SOUNDS = 6`
* `NUMBER_OF_VARIABLES = 11`
* `NUMBER_OF_BYTES = 12`
* `NUMBER_OF_PRESETS = 6`
* `NUMBER_OF_BANKS = 9`
* `PRESET_SIZE = 84`

Every sound owns `NUMBER_OF_BYTES` slots inside `variable[NUMBER_OF_SOUNDS][NUMBER_OF_BYTES]`.  Values are not stored as single bytes; the code in `getVar`/`setVar` uses the `byteCoordinate` and `bitCoordinate` tables to place each parameter into a bitfield of the proper width from `variableDepth`.

```c++
const unsigned char variableDepth[] = {10, 7, 7, 7, 7, 8, 10, 10, 8, 7, 7};
const unsigned char byteCoordinate[] = {0, 1, 2, 3, 3, 4, 5, 7, 8, 9, 10};
const unsigned char bitCoordinate[]  = {0, 2, 1, 0, 7, 6, 6, 0, 2, 2, 1};
```

Because of the variable-width encoding a single entry can straddle up to three bytes.  `setVar` checks that by bumping `byteShift` to `1` or `2` as needed.

### Flags inside `SETTING`

`SETTING` (index 8) stores several booleans in a single byte.  The UI code names them as bit positions:

* `TUNED_BIT = 0`
* `LEGATO_BIT = 1`
* `REPEAT_BIT = 2`
* `SYNC_BIT = 3`
* `SHIFT_DIR_BIT = 4`

Bit math happens all over `UI.ino` and `SOUND.ino`.  When you add new toggles, make sure you update both the UI mask logic and any MIDI/engine code that depends on it.

## Presets in EEPROM

* EEPROM addresses `E_BANK = 1001` and `E_PRESET = 1002` remember the last selected location.
* Byte `1000` is a virgin-boot sentinel.  When it's not set the firmware plays `ZZ.WAV` automatically and then flips the flag.

Presets themselves live on the SD card, not EEPROM.  Saving writes the packed `variable` buffer to disk using the helper routines in `SD.ino`.

## Expanding the map

If you need more parameters:

1. Increase `NUMBER_OF_VARIABLES`.
2. Add the new label to `labels`.
3. Extend `maxValue`, `variableDepth`, `byteCoordinate`, and `bitCoordinate` with the correct widths.
4. Re-run the allocation math: make sure the last parameter fits within `NUMBER_OF_BYTES`.
5. Teach `UI.ino`, `SOUND.ino`, and any MIDI handlers about the new value.

It's finicky, but once you understand the bit math the firmware is surprisingly forgiving.
