# Setting up grblHAL

Not using the web builder (https://svn.io-engineering.com:8443/) to have more control over the configuration. I don't think it supports my usecase correctly.

This is partially based on https://github.com/grblHAL/ESP32

## Download ESP SDK

Can't use v4.3 here like https://github.com/grblHAL/ESP32 suggests, because that doesn't support esp32s3 (doesn't list it in `idf.py --list-targets`). Instead clone everything:
```sh
git clone https://github.com/espressif/esp-idf.git

cd esp-idf
```
Then choose ESP SDK version. S3 support (esp32s3) was added in 4.4.x, so I chose the latest 4.4.x which is 4.4.8.

Using the newest version doesn't work, because they changed the API. (v6.0 caused compilation errors that I couldn't figure out how to fix.)

List versions:
```sh
git tag
```
Select version:
```sh
git checkout v4.4.8
```

Important! Update submodules:
```sh
git submodule update --recursive --init
```

Must install this, otherwise `./install.sh` below will complain: (`virtualenv` isn't the same thing as `venv`)
```sh
pacman -S python-virtualenv
```

Lastly, install the SDK:
```
./install.sh
. ./export.sh
```

Note that `. ./export.sh` adds `idf.py` to PATH.

```sh
cd ..
```

## Build

Clone grblHAL for ESP32. This includes the `grblHAL/core` as a submodule.

This guide was tested on commit: c07026a885d9b0c2d34b8afd99baf3c748ee6699

```sh
git clone --recursive --shallow-submodules https://github.com/grblHAL/ESP32
cd ESP32
```

Patch `main/my_machine.h`:
* Uncomment: `#define BOARD_MKS_DLC32_MAX_V1`.
* Uncomment: `HOMING_PULLOFF_ENABLE`, `WEBUI_ENABLE`, `PROBE_ENABLE 0` (must uncomment to disable the probe, it defaults to enabled).
* Add: `#define SPINDLE0_ENABLE SPINDLE_PWM0_NODIR`.

Patch `main/boards/mks_dlc32_max_1_0_map.h` for our board:
* Comment out the pin that we reuse for "spindle enable": `// #define COOLANT_MIST_PIN        AUXOUTPUT3_PIN //Beeper header`
* Change `#define SPINDLE_ENABLE_PIN` to `AUXOUTPUT3_PIN` to use the beeper pin.
* Comment out: `CYCLE_START_PIN`, `LED_PIN`, `PROBE_PIN`, `SAFETY_DOOR_PIN`.

Build:

```sh
rm -rf build
# You can list available targets with `idf.py --list-targets`
idf.py set-target esp32s3
idf.py build
rm -rf build
```

## Runtime configuration of grblHAL

Boot the board, connect via Universal GCode Sender.

Set settings:

```
; --- Important:
; Enable hard limits
$21=1

; --- Steppers:
; Axis inversion mask. 2 = Y
$3=2

; Steps/mm XYZ
$100=320
$101=320
$102=320
; Max speed, mm/min XYZ
$110=2000
$111=2000
$112=1000
; Acceleration, mm/sec^2 XYZ
$120=75
$121=75
$122=25
; Max travel, mm
; Note that you must add $27 pulloff distance to those. And if bit +8 in $22 is NOT set, add $27 AGAIN.
; The true range of the machine is 270x200x180
$130=273
$131=203
$132=183

; --- Spindle:
; Max RPM
$30=24000

; --- Homing:
; Pulloff distance, mm
$27=3
; XYZ direction inversion. 1 = invert X.
$23=1
; Enable homing and set settings.
; +1 = enable
; +2 = enable per-axis homing commands
; +4 = require homing on startup
; +8 = set coords to zero instead of subtracting pullof distance
; +16 = Refuse to home from a held switch. This is required when both ends have switches wired to the same pin,
;         as opposed to only having the switch on ONE end and relying on soft limits for the other end.
;       If you don't set this and start homing when the OPPOSITE switch is held, GRBL will try to "pull off" it
;         in the direction opposite to homing, which will result in a crash.
;       Or, actually, it will abort after moving the pulloff distance, which might prevent a crash if your swithch if flexible enough,
;         but this is still bad.
; There are more options that I didn't need, see `main/grbl/settings.c`.
; NOTE: Setting this to values other than 1 confuses Unversal GCode Sender, so it grays out the Home button.
;         Then you must manually do $H to home (and if you set `+4`, the machine will remain locked otherwise).
$22=31
; Enable soft limits. This requires homing to be enabled first.
$20=1

; --- Misc:
; Clamp jogging with soft limits. Otherwise trying to jog out of bounds doesn't move at all, instead of clamping.
$40=1
```
Wifi settings:
```
; --- Wifi:
$74=network name
$75=password
; Note that $$ (at least in universal gcode sender) seems to print $74 wrong when spaces are involved. But this is purely a visual bug.
; Printing it directly with `$74` works fine.
You also need `$73=1` (wifi mode = "station", aka connect to an existing network), but that's the default value.

Then to get the IP address, run `$I` and look for `STA IP`.
```
