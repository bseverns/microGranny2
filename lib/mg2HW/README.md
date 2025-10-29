# mg2HW shim

The original Bastl firmware expects an `mg2HW` library that wraps all of the LED, display, and button handling for the microGranny 2 hardware.  Bastl released it as part of the firmware bundle, but it isn't mirrored in this repo.

Drop the official library sources here (the folder should expose `mg2HW.h` at `lib/mg2HW/mg2HW.h`) or point `lib_deps` at your own fork if you need to customise the hardware layer.

Without the real library the build will fail when PlatformIO tries to compile the project.
