# Eryone Thinker V2 Firmware
###  (Thinker SE: Marlin 2.1.2.9 · Thinker SV2: Marlin 2.0.x)

![GitHub Stars](https://img.shields.io/github/stars/Eryone/mega2560.svg)
![GitHub contributors](https://img.shields.io/github/contributors/Eryone/mega2560.svg)
![GitHub followers](https://img.shields.io/github/followers/Eryone.svg)

- Author: 
  - **Tom Yuan**
  - **Eryone** 


## Support 3D printer

| 3D Printer Model    | Motherboard                | Stepper Driver    
| ------------------- | -------------------------- | --------------
| Eryone Thinker SV2  | Thinker V2 Motherboard     | TMC2208
| Eryone Thinker SV2  | Thinker V2 Motherboard     | TMC2209
| Eryone Thinker SE   | Thinker V2 Motherboard     | TMC2208
| Eryone Thinker SE   | Thinker V2 Motherboard     | TMC2209

### Thinker V2 Motherboard

![](http://www.eryone.com/forum/download/file.php?id=433)

## Source               

- *[Stable Firmware](https://github.com/Eryone/mega2560)* This is the offical firmware, not have much feature but stable.
- *[Marlin officail](https://github.com/MarlinFirmware/Marlin)* Marlin offical firmware release main page
- *[Marlin Gcode](https://marlinfw.org/meta/gcode/)* Marlin offical gcode introduction

## Feature
- Thinker SV2/SE with TMC2208
  - **Filament change**
  - **Z babystep** 
  - **Mesh bed leveling or auto bed level**
  - **SD card print percent**
  - **LCD shows filament used**

- Thinker SV2/SE with TMC2209
  - **TMC UART**
  - **Sensorless homing**
  - **Filament change**
  - **Z babystep** 
  - **Mesh bed leveling or auto bed level**
  - **SD card print percent**
  - **LCD shows filament used**


## Thinker SE on Marlin 2.1.2.9

The Thinker SE configurations and hex files have been ported from Eryone's Marlin 2.0.x fork to
Marlin 2.1.2.9, the latest stable Marlin release (October 2026). Marlin 2.1.2.9 supports the Thinker V2
motherboard directly (`MOTHERBOARD BOARD_RAMBO_THINKERV2`, PlatformIO environment `rambo`). The Marlin
source in `THINKER V2/Marlin` is stock 2.1.2.9 with three small source changes (see *Thermal errors*,
*TMC2209 variants* and *Power-loss recovery* below); everything else specific to the Thinker SE is in the
configuration files.

> **Status:** every variant builds, and each one was compared setting by setting with the Eryone 2.0.x
> firmware it replaces. The new firmware has **not been tested on a printer yet**. Keep the previous hex
> file at hand (it is in the history of the `master` branch).

| Variant (folder in `THINKER V2/config/Eryone/THINKER SE/`) | Hex file (`THINKER V2/Hex File/THINKER SE/`) | Stepper drivers | Bed leveling | Power-loss recovery | Filament runout sensor |
| --- | --- | --- | --- | --- | --- |
| `Stock` | `THINKERSE.hex` | TMC2208 (standalone) | Manual mesh, 5×5 | Built in, off by default | – |
| `Stock with no power resume` | `THINKERSEnopowerresume.hex` | TMC2208 (standalone) | Manual mesh, 5×5 | No | – |
| `Stock with Bltouch` | `THINKERSEwithBltouch.hex` | TMC2208 (standalone) | BLTouch, bilinear 3×3 | Built in, off by default | – |
| `Stock with Bltouch   filament runout` | `THINKERSEwithBltouchfilamentrunout.hex` | TMC2208 (standalone) | BLTouch, bilinear 3×3 | Built in, off by default | Z-min port (pin 10) |
| `Stock with TMC2209 sensorless` | `THINKERSEwithTMC2209sensorless.hex` | TMC2209 (UART), sensorless X/Y homing | Manual mesh, 5×5 | Built in, off by default | – |
| `Stock with TMC2209 sensorless   Bltouch` | `THINKERSEwitTMC2209sensorlessBltouch.hex` | TMC2209 (UART), sensorless X/Y homing | BLTouch, bilinear 3×3 | Built in, off by default | – |
| `TMC2209 with filament runout` | `THINKERSETMC2209withFilamentrunout.hex` | TMC2209 (UART), sensorless X/Y homing | Manual mesh, 5×5 | Built in, off by default | LEVEL header (pin 4) |

As in Eryone's configurations, every variant also drives a second Z motor from the E1 driver socket.

### Before and after flashing

The update resets every stored setting (the EEPROM layout changed), so:

1. Before flashing, send `M503` and keep the output (E-steps, offsets, PID values, motor currents).
2. On the first start the LCD shows `Err: EEPROM Version` and offers to *Initialize EEPROM*: choose
   **Reset** (the same as `M502` followed by `M500`). *Ignore* runs on the defaults and asks again at every start.
3. Re-enter your custom values (for example `M92`, `M206`, `M301`, `M906`), then calibrate again: the BLTouch
   Z offset (`M851 Z…` or Z babystepping), the bed mesh (LCD *Level Bed*; on the BLTouch variants also `G28` then
   `G29`), optionally a hotend PID tune (`M303 E0 S210 C8 U1`), and save with `M500`. Power-loss recovery is
   off after the reset; see *Power-loss recovery* below if you want it.

### Differences from the Eryone 2.0.x firmware

- **Thermal errors (all variants).** On a heater or thermistor fault the printer switches the heaters off,
  beeps and halts where it is, with the error on the LCD, as Eryone's firmware did. Stock Marlin 2.1.2.9
  first lifts and parks the nozzle. On this board that move outlasts the 4-second watchdog, so the board
  rebooted and the error was never shown; and with upstream's fix for that
  ([MarlinFirmware/Marlin#28645](https://github.com/MarlinFirmware/Marlin/pull/28645)), a fault during a
  repeat homing makes the park drive an axis past its end. This tree leaves the park out instead: a
  Thinker SE change in `Marlin/src/module/temperature.cpp`, not taken from upstream.
- **Mesh bed leveling variants** (`Stock`, `Stock with no power resume` and the two TMC2209 variants without
  BLTouch): `G29 S1` sent from a host or terminal no longer homes first. Send `G28` before it. The LCD
  *Level Bed* procedure still homes by itself.
- **BLTouch variants.** Marlin 2.1 requires *Z safe homing* with a probe. `G28` homes as before. A lone
  `G28 Z` (or LCD *Home Z*) now homes X and Y first if they have not been homed since power-on; otherwise it
  first moves the probe to the Z homing point (X40 Y10, the spot Eryone's firmware probed after homing;
  `Stock with TMC2209 sensorless   Bltouch`: X60 Y60).
- **Z babystepping** (`Stock with Bltouch`, `Stock with TMC2209 sensorless   Bltouch`). One click of the knob
  now moves Z by one motor step (0.0025 mm), as in the other variants. Eryone's configurations for these two
  set `BABYSTEP_MULTIPLICATOR_Z` to 0.5, which moved 0 steps per click.
- **TMC2209 variants.** The Thinker V2 board connects only the send line of the drivers' UART, so Marlin 2.1
  cannot read the drivers back; the motor currents, stealthChop and the sensorless-homing sensitivity are
  still written. As in Eryone's firmware, the `TMC CONNECTION ERROR` message this causes on the LCD is
  hidden: a Thinker SE change in `Marlin/src/feature/tmc_util.cpp`, switched on by `TMC_UART_IS_WRITE_ONLY`
  in the three TMC2209 configurations. The serial log still shows the failed connection test at start-up
  and after `M122`. Because the drivers cannot be read back:
  - X and Y return to stealthChop after every homing. If you use spreadCycle (`M569 S0 X Y`), send it again
    after `G28`.
  - Do not enable `HYBRID_THRESHOLD` or `TMC_HOME_PHASE` unless the RX line (D19) is wired: both need to read
    the drivers, and `HYBRID_THRESHOLD` breaks sensorless homing. If you do wire RX, also delete
    `#define TMC_UART_IS_WRITE_ONLY` (the last line of `Configuration_adv.h`) so that a real TMC connection
    error is shown on the LCD again.
- **`TMC2209 with filament runout`.** The runout sensor on the LEVEL header (pin 4) is active. Eryone's hex of
  the same name was built without runout support. If nothing is plugged into that header, every print pauses
  for a filament change as soon as it starts. Plug the sensor in, turn it off with `M412 S0` and `M500`
  (`M502` turns it back on), or use the `Stock with TMC2209 sensorless` firmware instead.
- **Power-loss recovery** is built into every variant except `Stock with no power resume`, but it is now
  **off by default** (Eryone's firmware had it on). Marlin 2.1.2.9 decides differently when to save the
  recovery file. With a bed mesh active, on parts of the bed where the mesh is below -0.05 mm it rewrites the
  file on every printing move, which can make SD prints stutter on detailed curves. Where the mesh is about
  one layer height or more above zero it may not update the file, and a resume then starts from an older
  layer. Even with an up-to-date file, a resume with a bed mesh active can come back slightly too high or
  too low, by the mesh height difference between the resume point and the X/Y home position. This is
  upstream Marlin behaviour, not fixed upstream yet, and resuming has not been tested on a Thinker SE. To use
  recovery anyway, switch it on with `M413 S1` and `M500` (or LCD *Configuration* > *Power Outage*, then
  *Store Settings*), watch the first layers after a resume, and switch it off again with `M413 S0` and
  `M500` if prints stutter or a resume goes wrong. One related upstream fix is included: stock 2.1.2.9 rounds Z to
  0.1 mm when it resumes, and this tree keeps three decimals as Eryone's firmware did
  ([MarlinFirmware/Marlin#27965](https://github.com/MarlinFirmware/Marlin/pull/27965), in
  `Marlin/src/feature/powerloss.cpp`).
- **Pausing.** `M600` and the runout sensor park as before (20 mm lift, 450 mm unload). `G27` (park) now also
  lifts the nozzle by at least 20 mm.
- **LCD.** The status screen alternates the print percentage and the elapsed time on one line.

### Values kept from Eryone's configurations that are worth checking

- **Probe offset of `Stock with TMC2209 sensorless   Bltouch`:** X40 Y-30, as in Eryone's hex and in all of
  Eryone's TMC2209 configurations with a probe (Thinker SV2 included), while Eryone's BLTouch configurations
  without TMC2209 use X40 Y10. Measure where your probe is and set `M851 X… Y…`, then `M500`.
  With Y-30 the probe can only reach Y 10–255 of the bed.
- **Coordinate frame of the TMC2209 variants:** homing sets X10 Y-25 and the travel is X10–310 / Y-25–285, as in
  Eryone's TMC2209 hex files. If prints are not centred on the bed, adjust with `M206 X… Y…` and `M500`.
- **Z homing point of `Stock with Bltouch` and `Stock with Bltouch   filament runout`:** X40 Y10 for the probe,
  which puts the nozzle at X0 Y0. With a probe offset larger than X40 or Y10, `G28` cannot home Z and shows
  *Z Probe Past Bed*; then change `Z_SAFE_HOMING_X_POINT` / `Z_SAFE_HOMING_Y_POINT` in `Configuration.h`.
- **Filament unload length:** 450 mm in every variant, as in all of Eryone's hex files. The configuration text of
  `Stock with Bltouch   filament runout` and `Stock with TMC2209 sensorless   Bltouch` said 25 and 26 mm, but no
  released hex was built with those values. Change it with `M603 U<length>` and `M500` if your setup needs less.

### Thinker SV2

The Thinker SV2 configurations (`config/Eryone/THINKER SV2/`) and hex files (`Hex File/THINKER SV2/`) are still
Eryone's Marlin 2.0.x versions. They do not build with the Marlin 2.1.2.9 tree; the Marlin 2.0.x tree is in the
history of the `master` branch.

## Build and Install
- **Please ues the "M502" and "M500" to reset the EEPROM when you uploaded the firwmare**
We prepared the hex file, you can choose the correct one then upload it to your printer.
Recommand to use the **Ultimaker Cura** upload it.
Refer to this link:
- *[Upload firmware via Cura](https://www.youtube.com/watch?v=SpdjvYkUQY4)*

To build the Thinker SE firmware (Marlin 2.1.2.9) you'll need [VS Code](https://code.visualstudio.com/) with the [PlatformIO](https://docs.platformio.org/en/latest/integration/ide/vscode.html) extension. Open the "THINKER V2" folder: "platformio.ini" already selects the "rambo" environment that the Thinker V2 motherboard needs, and the "Marlin" folder already contains the **Stock** configuration. Detailed build and install instructions at:

- *[Youtube Guide Video](https://www.youtube.com/watch?v=b2D4I9Yxejw)*
- *[Installing Marlin (PlatformIO with VS Code)](https://marlinfw.org/docs/basics/install_platformio_vscode.html)*
- **Please ues the "M502" and "M500" to reset the EEPROM when you uploaded the firwmare**

### Change the language: Open the "configuration.h" then find this line:
- "#define LCD_LANGUAGE" then set the language you want to set. Built it and upload it to your printer.

## How to use the config&Guide
- Notes: Before this section, if you are not familiar with the marlin then you should back to the previous section.
- Go to "Thinker V2"→"docs" to refer the guide of TMC2209 with Thinker V2 motherboard
- For the Thinker SE on Marlin 2.1.2.9, use that guide only for the hardware setup (jumpers, DIAG wires) and the calibration G-codes: its *Firmware* part was written for Eryone's Marlin 2.0 package, and the TMC2209 folders in "config"→"Eryone"→"THINKER SE" already contain those settings.
- Open the ”config“→“Eryone”→Then choose the correct model folder then enter
- Copye the ** "configuration.h" and "configuration_adv.h" **
- Then paste them to ** "Marlin" ** folder
- Only the "THINKER SE" folders build with this Marlin 2.1.2.9 tree. For a Thinker SV2, flash a file from "Hex File"→"THINKER SV2", or build its configuration with the Marlin 2.0.x tree in the history of the `master` branch (see *Thinker SV2* above). Do not convert an SV2 configuration by following the compiler errors: the SV2 files select `MOTHERBOARD BOARD_RAMBO`, which in Marlin 2.1.2.9 uses the stock RAMBo pins instead of the Thinker V2 pins.
- **Please ues the "M502" and "M500" to reset the EEPROM when you uploaded the firwmare**


## Join our group& Contact us
- *[Facebook](https://www.facebook.com/groups/247271792709370/)*
- *[Instagram](https://www.instagram.com/eryone3d/)*
- *[Eryone Forum](https://www.instagram.com/eryone3d/)*
- *[Youtube](https://www.youtube.com/eryone3d)*














