Standalone KUSBA resonance measurement for Prusa MINI / MINI+
This is an experimental community workflow for measuring a Prusa MINI's real mechanical
resonances with a KUSBA USB accelerometer while leaving the printer on Prusa Buddy firmware.
No Klipper conversion is required.
It reproduces the workflow used on a heavily modified MINI nicknamed "Filbert".
What this kit contains
Exact tested KUSBA firmware source for our white-dot / ISSI-flash KUSBA
ADXL345 self-test script
Windows CSV capture script
X unshaped ringdown G-code
Y unshaped ringdown G-code with the bed heater explicitly disabled
AI analysis prompt templates
Hardware links
KUSBA Amazon listing used in our experiment:
https://a.co/d/045nkkEG
Official KUSBA project:
https://github.com/xbst/KUSBA
M6 nozzle mount:
https://github.com/xbst/KUSBA/blob/main/Mounts/M6_KUSBA_Mount.stl
KUSBA documentation:
https://docs.isiks.tech/
The official M6 mount is intended for KUSBA v2.3/v2.4 and calls for an M6 x 10 mm
SHCS/BHCS after removing the nozzle. In our MINI setup the mount hung below the normal
nozzle height, so we ALWAYS homed before installing the nozzle-mounted sensor.
IMPORTANT firmware compatibility note
The included PlatformIO project is the exact build that worked on our KUSBA with the
ISSI flash chip. Rampon documentation notes that some KUSBA batches use an ISSI
IS25LP016D instead of the Winbond flash. The ISSI chip can be identified by a white
dot/marking on the large 8-pin flash chip.
This project uses:
boot2_is25lp080_4_padded_checksum.S
Do not assume that boot2 is correct for every KUSBA revision/flash chip.
If your KUSBA is not the white-dot ISSI variant, use the source as a reference and adapt
the flash boot2 appropriately rather than blindly flashing it.
Windows setup
Install:
Python 3
Git for Windows
PlatformIO and pyserial:
py -m pip install -U platformio pyserial
Extract the firmware project under:
firmware/Filbert_KUSBA_ADXL_AutoTest_ISSI_DIV4/
Build it by running:
    build_firmware.bat

The UF2 should appear at:
    .pio\build\kusba_issi\firmware.uf2

Flashing the RP2040
Disconnect the KUSBA.
Hold BOOT while plugging it into USB so it appears as an RPI-RP2 drive.
Copy firmware.uf2 onto the RPI-RP2 drive.
The board reboots and should appear in Windows as a USB Serial Device (COMx).
The COM number can change after reflashing.
Verify the sensor before mounting it
From the firmware project folder:
    python adxl_autotest.py

Select the KUSBA COM port.
A healthy unit should report approximately:
    devid=0xE5
    bw_rate=0x0E
    data_format=0x0B
    power_ctl=0x08
    sample rate around 1600 samples/s
    stationary gravity magnitude around 1 g

Do not proceed if DEVID is not 0xE5.
Recording
Use:
    python tools\kusba_capture.py

Choose the KUSBA COM port, give the run a label, and use 35 seconds for a baseline.
The capture script writes CSV with:
    time_s,device_us,x_raw,y_raw,z_raw

X measurement
Use:
gcode/01_X_unshaped_baseline.gcode
The nozzle-mounted KUSBA is NOT installed while the MINI homes. The file:
turns both heaters off
homes normally
parks at X90 Y90 Z90
pauses
you install the KUSBA and arrange cable slack
start the PC capture
resume the MINI
X input shaping is disabled
repeated X60 <-> X120 moves excite ringdown
remove the KUSBA at the final pause
Repeat the baseline test twice without changing belts/mounting.
Y measurement
Use:
gcode/02_Y_unshaped_baseline_COLD.gcode
For Y, rigidly secure the KUSBA to the moving bed using an electrically insulating
mounting method. Do not let exposed electronics short against metal or heater conductors.
This test:
commands M140 S0
contains NO M190
commands M104 S0
homes X and Y only
never homes or moves Z
disables Y input shaping only
excites repeated Y60 <-> Y120 ringdowns
Repeat twice.
Analysis and shaper selection
See:
docs/AI_ANALYSIS_PROMPTS.md
The workflow is:
Repeat the unshaped baseline twice.
Use FFT/PSD of many post-stop ringdowns to find repeatable mechanical modes.
Do not simply set the shaper equal to the tallest resonance.
Compare MZV, ZVD and EI by measuring the vibration that remains after applying them.
Fine-sweep the best shaper's frequency.
Enter only the final type + Hz into the MINI UI.
Our result on "Filbert"
This is an example, NOT a value to copy to another machine.
X physical modes:
primary driven-axis mode about 129 Hz
important coupled mode about 87 Hz
Final X setting:
EI / 108 Hz
Y physical modes:
primary bed mode about 60 Hz
coupled family roughly 46-53 Hz
Final Y setting:
EI / 52 Hz
The important result was that the best shaper frequency did NOT necessarily equal the
largest physical resonance peak. Directly measuring residual vibration after applying
candidate shapers gave better answers.
Safety / limitations
This is experimental, not an official Prusa or Isik's Tech procedure.
Check cable slack before every motion test.
Do not home with the M6 nozzle KUSBA mount installed if it hangs below the nozzle.
Keep the bed cold during the under-sheet/bed-mounted Y test.
Do not allow a bare PCB to short against metal or heater conductors.
The supplied motion test uses 160 mm/s and 4000 mm/s^2 acceleration. If your machine
is not mechanically sound or you are uncomfortable with that acceleration, reduce it.
Power-cycle the MINI or restore your saved Input Shaper settings after unshaped tests.
Values are machine-specific. Do not copy EI 108 / EI 52 without measuring your own MINI.
