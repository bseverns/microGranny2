# microGranny2 Field Manual (Fork Edition)

Welcome to my annotated fork of the Bastl Instruments **microGranny 2.0** firmware.  The goal here isn't to rewrite Vaclav's code (yet) – for now, it's to document the living daylights out of it so more people can mod, repair, and teach with this tiny granular beast.  Think of this repo as half studio notebook, half roadie handbook.

## Why this fork exists

* The original firmware is legendary, but the documentation lives in forum posts, PDF scans, and memories of modular nerds.  This fork collects the most useful breadcrumbs in one place.
* Most changes so far are documentation + build tooling.  The only behavior change in this fork is a small MIDI tweak: randomize-by-CC now triggers the same UI feedback and knob resync you get from the front panel.
* Every note below comes from reading the code that ships in this repository.  If it isn't in these `.ino` files, it isn't promised here.

## Fork status (snapshot)

**Completed in this fork**

* PlatformIO scaffolding + `src/` reorg so the firmware builds outside the Arduino IDE.
* Documentation additions: parameter packing notes, PlatformIO walkthrough, and this annotated README.
* `mg2HW` shim README to document the missing hardware layer dependency.
* MIDI randomize CC feedback (see `src/MIDI.ino` in `proceedCC()`).

**Open work / next steps**

* Bring the official `mg2HW` library into `lib/mg2HW/` (or wire `lib_deps` to a fork) so PlatformIO builds without manual copy steps.
* Document the LED/color language and UI message patterns once `mg2HW` references are in hand.
* Cross-check the MIDI randomize-by-CC feedback on real hardware and document any quirks or regressions vs upstream.
* Decide whether to pin exact SdFat/WaveRP versions in `platformio.ini` (or document known-good versions) to avoid build drift.

## Hardware + tooling checklist

You'll want the same setup Bastl used:

* **Controller board:** Bastl microGranny 2.0 (ATmega328P at its heart, Arduino Uno-compatible toolchain).
* **Audio + UI shield:** `mg2HW` library (not bundled here; grab it from the original Bastl release).
* **Storage:** FAT-formatted microSD card.
* **Libraries:**
  * [SdFat](https://github.com/greiman/SdFat) – filesystem access.
  * [WaveRP](https://code.google.com/archive/p/waverp/) – WAV streaming/recording.
* **Tooling:** Arduino IDE (1.8.x era) or Arduino CLI with the AVR toolchain.

Flash routine:

1. **PlatformIO heads-up:** the repo now ships with a ready-to-run `platformio.ini`.  Install PlatformIO, clone the repo, and run `platformio run` to build or `platformio run --target upload` to flash.  Drop Bastl's `mg2HW` sources under `lib/mg2HW/` (or tweak `lib_deps`) so the hardware layer resolves.
2. **Arduino IDE traditionalists:** install the SdFat and WaveRP libraries, park `mg2HW` in your libraries folder, then open `src/microGranny2.ino` and upload as usual.  The IDE still slurps in the companion `.ino` tabs because they're sitting right next to the main sketch inside `src/`.

## Repo map

| File | What it holds |
| --- | --- |
| `src/microGranny2.ino` | Bootstraps the hardware, SD card, MIDI, and memory state.  Think of it as the conductor orchestrating the other modules. |
| `src/SOUND.ino` | Playback engine: loop logic, grain triggering, envelope shaping, and keeping the WaveRP stream on leash. |
| `src/MEM.ino` | Compact bit-packed parameter storage for six sounds × multiple presets and banks.  Also contains the parameter metadata tables documented below. |
| `src/MIDI.ino` | MIDI parser + note buffer.  Handles legato, sustain, clock sync, and sample-rate tweaks via a 49-entry lookup table. |
| `src/SD.ino` | Disk ops: SD initialisation, file indexing, sample playback, and the record-to-WAV routine. |
| `src/UI.ino` | Front-panel choreography – LEDs, RGB feedback, display messages, long-press/shift combos, and record mode UI. |
| `src/fileNames.ino` | Utility helpers for naming and listing samples on the card. |

## Parameter cheat-sheet (straight from `MEM.ino`)

The firmware stores eleven variables per sound, bit-packed inside `variable[NUMBER_OF_SOUNDS][NUMBER_OF_BYTES]`.  Here's the layout pulled directly from the tables in `MEM.ino`:

| Index | Label (`labels`) | Meaning | Range (`maxValue`) | Notes |
| --- | --- | --- | --- | --- |
| 0 | `r` | Sample rate | `0–1023` | Higher is faster; tuned playback uses the note table in `MIDI.ino`. |
| 1 | `c` | Bit-crush depth | `0–127` | Drives WaveRP's `setCrush` controls. |
| 2 | `a` | Attack | `0–127` | Envelope generator attack time. |
| 3 | `r` | Release | `0–127` | Envelope generator release time. |
| 4 | `g` | Loop length | `0–127` | Tied to `loopLength` in `SOUND.ino`. |
| 5 | `m` | Shift speed | `0–255` | Signed rate for granular shifting (see `shiftSpeed`). |
| 6 | `s` | Start position | `0–1023` | Sample start index. |
| 7 | `e` | End position | `0–1023` | Sample end index. |
| 8 | (none) | Mode/flags (`SETTING`) | `0–63` | Bitfield for tuned/legato/repeat/sync/random shift toggles. |
| 9 | (none) | Sample name char 1 | ASCII `0–127` | First stored character of the SD filename. |
| 10 | (none) | Sample name char 2 | ASCII `0–127` | Second stored character of the SD filename. |

Those values load and save through `loadPreset`, `savePreset`, `getVar`, and `setVar`.  Want to add a new parameter?  Update `NUMBER_OF_VARIABLES`, extend `labels`, `maxValue`, `variableDepth`, and the byte/bit coordinate tables, then teach the UI and storage code about it.

## MIDI behaviour crib notes

* **Buffering:** `MIDI.ino` keeps a 16-slot FIFO (`midiBuffer`).  Duplicate notes get deduped before insertion.
* **Legato:** If `legato` is true and the incoming MIDI note is between 23 and 65, the code reuses the active voice and simply retunes playback via the `noteSampleRateTable`.
* **Clock sync:** `clockCounter` increments with MIDI clock messages; `instantLoop` routines in `SOUND.ino` use it to align stutter loops.
* **Sustain:** `sustain` gates note-off handling so you can latch grains with a pedal.

## SD card rituals

* `initSdCardAndReport()` is strict: it wants a card that passes `Sd2Card::init` and exposes a FAT16/32 root.
* The playback path caches directory positions per sound inside the `index[]` array.  Once a sound slot has touched a file, subsequent plays re-open by index instead of name – faster, but it means renaming files on the card without rebooting can confuse things.
* Recording drops 16-bit WAVs named `<bank><preset>.WAV` (e.g. `30.WAV`).  The naming math lives in `trackRecord()`; study it before changing your folder structure.

## Hacking roadmap

1. **Audit parameters** – start in `MEM.ino` so you know what space is left in `variable[]` and the EEPROM layout (`E_BANK`, `E_PRESET`).
2. **Touch UI intentionally** – `UI.ino` manages long presses, combo buttons, and the 7-seg display text.  Update the helper strings and knob mappings in the same breath.
3. **Mind the wave player** – anything touching `wave` (WaveRP) should pause/resume carefully; see `renderLooping()` and `renderGranular()` in `SOUND.ino` for the existing handshake.
4. **Keep MIDI light** – the parser runs in the main loop; avoid blocking operations there or you'll miss clocks.

## Session notebook

* ✅ EEPROM boot guard: on first boot `EEPROM.read(1000)` forces playback of `ZZ.WAV`, then resets the init flag.  Handy for smoke-testing new builds.
* ✅ `RECORD_RATE` is hard-coded to 22,050 Hz.  Raise it if you have the bandwidth, but watch `MAX_FILE_SIZE` (currently 100 MB) and SD card performance.
* ✅ `instantLoop` mode `2` is the synced stutter.  `instantClockCounter` is set elsewhere to quantise the loop restarts.
* ❓ TODO: Document the full color/LED language once the `mg2HW` library is in hand.

Pull requests are welcome if you discover quirks or want to teach the next hacker something new.  Keep it punk, but keep it accurate.

## Toolchain notebook

* `docs/platformio.md` walks through the PlatformIO setup, dependency dance, and the new folder layout.
* `docs/status.md` keeps a short ledger of what this fork has changed and what is still pending.

## Fork changelog (high level)

* 2026-01-28 — Documentation refresh, including fork status ledger and clearer orientation. (`077febb`)
* 2025-11-09 — MIDI randomize-by-CC now triggers UI feedback + knob resync. (`25eade7`)
* 2025-10-28 — PlatformIO scaffolding, `src/` reorg, and initial documentation drop. (`9ba1bcb`, `6f49849`)
