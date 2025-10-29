# PlatformIO build notebook

The OG Bastl sources were arranged for the Arduino IDE.  This fork keeps those `.ino` files intact but wires them into a PlatformIO environment so you can build, upload, and lint the firmware without touching the Arduino GUI.

## Project layout

PlatformIO expects the firmware entry point under `src/`.  Every `.ino` file from the original release now lives there so the automatic preprocessing still happens exactly like the IDE would do it.  You can still open the project in the Arduino IDE if you really want to – PlatformIO will just ignore the extra metadata files.

```
microGranny2/
├── platformio.ini          # PlatformIO configuration + dependency hints
├── lib/
│   └── mg2HW/              # Drop Bastl's mg2HW hardware library in here
├── src/
│   ├── microGranny2.ino    # Main sketch (same filename, new home)
│   ├── MEM.ino
│   ├── MIDI.ino
│   ├── SD.ino
│   ├── SOUND.ino
│   ├── UI.ino
│   └── fileNames.ino
├── docs/
│   ├── parameter-map.md
│   └── platformio.md
└── README.md
```

## Installing dependencies

Two external libraries are required:

* **SdFat** – available from the PlatformIO registry.  The `lib_deps` entry in `platformio.ini` pulls in a modern release automatically.
* **WaveRP** – Bastl based their granular engine on William Greiman's WaveRP hacks for the Adafruit Wave Shield.  The PlatformIO config references the GitHub mirror Bastl linked to in their documentation.

The microGranny also leans on Bastl's **mg2HW** helper library.  Because it isn't published in the PlatformIO registry, you need to supply it yourself by cloning the official sources into `lib/mg2HW` or pointing `lib_deps` at a fork.

```
# Example: pulling Bastl's hardware layer straight into lib/
git clone https://github.com/bastl-instruments/microGranny_hw.git lib/mg2HW
```

## Building and uploading

Once the dependencies are on disk, the usual PlatformIO workflow works out of the box:

```bash
# build the firmware
platformio run

# upload it to an Arduino-compatible board (set the port if auto-detect fails)
platformio run --target upload

# watch the serial output
platformio device monitor --baud 115200
```

The default environment targets an Arduino Uno-compatible ATmega328P, which matches the stock Bastl build.  If you're running a custom board, edit the `board` setting in `platformio.ini` to point at your variant.

## Bonus: keep both toolchains

Need to keep the Arduino IDE flow for a workshop or an older laptop?  No worries.  PlatformIO doesn't touch the source format; you can still open `src/microGranny2.ino` in Arduino IDE 1.8.x.  Just remember to keep the companion `.ino` files in the same folder so the IDE treats them as tabs.
