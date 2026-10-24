How to use the filament runout sensor for your printer

Note:
Please plug your filament runout sensor to z_min port
Marlin 2.1.2.9 (BOARD_RAMBO_THINKERV2) uses the Z_min port (pin 10) for the filament runout sensor
when a BLTouch is installed, so no pins file needs to be copied

Step:
1. Copy the config then replace the old config of the 'Marlin' folder
2. Upload the firmware via VSC or Arduino
3. Use the 'M502' and 'M500' to reset the EEPROM
4. Test it: M119 shows "filament: TRIGGERED" with filament loaded and "filament: open" without it.
   M600 tests the filament change itself


PS: This config based on Eryone Thinker SE printer stock endstop part.
If you find your sensor didn't work , check your sensor is "pull up" or "pull down".
Also, the sensor logic should be correct. In Marlin 2.1.2.9 it is FIL_RUNOUT_STATE in Configuration.h:
HIGH (this config) means the pin is HIGH when there is no filament; use LOW for a sensor with the
opposite logic. Pull-up / pull-down are FIL_RUNOUT_PULLUP (enabled) / FIL_RUNOUT_PULLDOWN.
Leave FIL_RUNOUT_ENABLED_DEFAULT at true: it switches the sensor on or off (M412)

Refer to this video to get more details of filament runout sensor:

author:
Chris Riley

https://www.youtube.com/watch?v=tl4VZ4zlxFQ&t=3s