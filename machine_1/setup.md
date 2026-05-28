## Building

Get Arduino Uno (knockoff is ok), and `CNC Shield 3.0`. Also three drivers, one per axis; I used DRV8825.

Connecting arduino must show up in `dmesg -w`, regardless of drivers. If not, yours is borked (or USB<->Serial chip needs flashing?).

### Modify the shield!

Must swap pins "spindle enable" and "Z limit". The correct pin order is: "X limit", "Y limit", "spindle enable", "Z limit". That's because new GRBL wants hardware PWM for the spindle control, which only exists on that pin.

### Configure and attach the NVBDL+ driver

After you're done with manually playing with the driver, you need to configure for control-by-wire.

Push SET to enter configuration. Select parameter with / and \\, start editing with SET, edit with / and \\, finish editing with SET, then exit configuration by selecting to Po and pressing SET.

* P0 = 1 - enable pwm control
* P1 = 1 - enable start by wire
* P3 = 1 - disable pwm inversion?

  The rest can stay at their default values (that is, the min available values for each).

NOTE: P3 is important! Failure to set this will enable the spindle by default when Arduino is attached, which is scary. If you did that, disconnect the control plug from the driver to stop it from spinning.

Wire GRBL's "spindle enable" to PWM, "spindle dir" to CW, and ground to ground. (Note: spindle enable and spindle dir pins are further from the board edge. Closer to the edge are ground pins, where you can connect "ground".)

Don't forget to compile-time configure GRBL for this setup, see below.

### Set up microstepping

<s>Major lack of microstep can cause jitter and lost steps? Minor lack of microstepping can cause noise?</s> Maybe that was due to overcurrent?

1/16 microstep looks ok.

Per axis, by connecting M1/M2/M3 pins on the CNC shield. See the driver datasheet.

For DRV8825, that's:

M2|M1|M0|Meaning
--|--|--|--
0 |0 |0 |Full step (2-phase excitation) with 71% current
0 |0 |1 |1/2 step (1-2 phase excitation)
0 |1 |0 |1/4 step (W1-2 phase excitation)
0 |1 |1 |8 microsteps/step
1 |0 |0 |16 microsteps/step
1 |0 |1 |32 microsteps/step
1 |1 |0 |32 microsteps/step
1 |1 |1 |32 microsteps/step

### Set up current

Failing to do this can cause overheating.

Read the desired amps from the stepper motor datasheet. You can decrease it a little if it looks too much (datasheet for DRV8825 says it's 2.5 amp MAX, or 1.5 amp without the heat sink).

#### For DRV8825:

Divide by two to get the voltage in volts. You should see that voltage between the big fat `-` on the CNC shield (aka the common ground?) and little metal knob on the driver that you're going to turn.

Do that with everything attached and powered up.

NOTE!! Rotating the knob clockwise DECREASES voltage!

### End stops

Those are per axis. The board has separate +X and -X (same for Y, Z), but the two are OR'ed.

Stops are off by default, on when hit. Then you wire them separately to +X, -X.

But it's better to invert them to stop on wire damage too. Invert by setting $5 to true. Then write them both in series to the SAME PIN, either +X or -X, doesn't matter.

#### Low-pass filter

The end stops might trigger when you enable the spindle. Then it might help to solder some capacitors over them.

[This link](https://github.com/grbl/grbl/issues/648) suggests 100 nF capacitor for 1k resistor, but the CNC shield v3 uses 10k resistor already.

I just soldered some junk big capacitors, and they seem to have worked. Mind the polarity!

## Uploading GRBL to arduino

Manual is here: https://github.com/gnea/grbl/wiki/Compiling-Grbl

* `sudo usermod -a -G dialout $USER`. Then log out and back in.

* Download Arduino IDE: https://support.arduino.cc/hc/en-us/articles/360019833020-Download-and-install-Arduino-IDE (Ubuntu package is wonky)

* Launch IDE. If it adds you to some group, you need to log out and back in before continuing.

* Download sources from: https://github.com/gnea/grbl/releases/tag/v1.1h.20190825

* Modify `config.h`:

  * I recommend enabling `HOMING_FORCE_SET_ORIGIN`. When homing, this will set (0,0,0) AFTER moving away from limit switches, as opposed to before.

    Otherwise the positive soft limit will be right at the limit switch, which for me meant that the switch was getting hit despite having the soft limit.

    If you invert some homing directions, this will ALSO give you a weird coordinate system, probably better to not do that.

  * For the NVBDL+ driver? enable `USE_SPINDLE_DIR_AS_ENABLE_PIN` and enable `INVERT_SPINDLE_ENABLE_PIN`.

* Zip the `gbrl` directory. (Note, it seems you can't rename this directory, hmm.)

* Sktech -> include library -> ZIP

* Examples -> grblUpload, and upload it.

## Using software

### Install BCNC

It directly controls the CNC, and has minimal CADding capabilities.

Download at https://github.com/vlachoudis/bCNC (Ubuntu package `bcnc`)

### Install FreeCAD

Also enable extra features in the settings, under `Path -> ...`. Those will only show up after you open the Path view.

If something doesn't work there, do `sudo apt install python3-pip`, `pip3 install opencamlib`.

### Configure controller

You can configure from GUI by going to CAM -> Controller, changing parameters, then pressing `Controller`.

Sometimes the settings might not apply the first time, run `$$` to check and press `Controller` again if needed.

Our current settings are below (as printed by `$$`). You should be able to paste them to the terminal to apply them.

```
$0=10
$1=25
$2=0
$3=4
$4=0
$5=1
$6=0
$10=1
$11=0.010
$12=0.002
$13=0
$20=1
$21=1
$22=1
$23=0
$24=25.000
$25=500.000
$26=250
$27=3.000
$30=12000
$31=720
$32=0
$100=1600.000
$101=1600.000
$102=1600.000
$110=500.000
$111=500.000
$112=500.000
$120=10.000
$121=10.000
$122=10.000
$130=154.000
$131=154.000
$132=65.000
```





## G-code

Links:

* GRBL-specific:
  * [Commands](https://github.com/gnea/grbl/blob/master/doc/markdown/commands.md)
  * [List of supported G-codes](https://github.com/gnea/grbl/blob/master/README.md) (scroll to the end)
  * [Error codes](https://github.com/gnea/grbl/blob/master/doc/csv/error_codes_en_US.csv)
* G-code
  * [Overview](http://linuxcnc.org/docs/stable/html/gcode/overview.html)
  * [G codes](http://linuxcnc.org/docs/stable/html/gcode/g-code.html#_g_code_quick_reference_table_a_id_gcode_quick_reference_table_a)
  * [M codes](http://linuxcnc.org/docs/stable/html/gcode/m-code.html#cha:m-codes)

Useful commands:


* `$RST=$` - reset settings (should restart bcnc after, I think?)
* `$RST=*` - reset settings and all state (should restart bcnc after, I think?)

* `$J=G91 X10 F80` - move X+=10mm at feed rate (speed) 80. Can combine multiple axes, and add `-` for negation: `$J=G91 Y-2 Z3 F80`
