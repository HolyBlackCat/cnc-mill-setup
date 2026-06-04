Tested on FluidNC 4.0.3.

Go to https://github.com/bdring/FluidNC, download `fluidnc-...-posix.zip` if you're on Linux.

Run `./install-wifi_s3.sh` to install the firmware. When it asks for the device, it means it's done and is trying to autorun `./fluidterm.sh` for you (which is a terminal to control the board), you can just Ctrl+C at this point.

There's also `install-bt`, that's for Bluetooth instead of WiFi, but they don't have a bluetooth version for S3, so alas. Trying to run this with S3 will say that it can't find a (suitable?) board.

Run `./install-fs_s3.sh` to upload the default filesystem, which includes the config file and web UI. This is independent from the firmware, don't reupload this when changing firmware or you'll lose your config.

Run `./fluidterm.sh` to connect to the board. Use...

```
$Sta/SSID=.....
$Sta/Password=.....
```
...to connect it to your WiFi.

The board can act either as a WiFi hotspot or connect to an existing one. This connects it to an existing one.

The devs recommend connecting to an existing WiFi, since the hotspot mode can crash the board after a while. It's less convenient too.

Connect to the board from your browser using the IP that it prints to `fluidterm.sh`.




## Wiring quirks:

There is a jumper that switches "Beeper" between 5v and VIN (input voltage). It's set to VIN by default, but we want to change it to 5v to control the spindle relay from it.
