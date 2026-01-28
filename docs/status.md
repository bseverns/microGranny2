# Fork status ledger

Use this page as the short, factual status board for this fork.  It is intentionally brief so new contributors can understand what changed vs upstream and what still needs attention.

## What this fork has already done

* Added PlatformIO scaffolding (`platformio.ini`) and moved the `.ino` files under `src/` so the codebase builds in PlatformIO without altering the Arduino IDE flow.
* Wrote documentation for the parameter packing layout and the PlatformIO setup.
* Added a `lib/mg2HW/README.md` shim to explain the missing hardware dependency.
* Changed MIDI randomize-by-CC to show UI feedback and resync frozen knobs (`proceedCC()` in `src/MIDI.ino`).

## What is still pending

* Import the official `mg2HW` sources (or point `lib_deps` at a maintained fork) so the PlatformIO build is reproducible without manual copying.
* Document the LED/color language and UI message patterns once the `mg2HW` code is available to reference.
* Validate the fork’s MIDI randomize CC behavior on hardware and document any behavioral differences vs upstream.
