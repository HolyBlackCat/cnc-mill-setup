# How to use the FabexCNC plugin in blender

## Download FabexCNC

Download FabexCNC from https://github.com/vilemduha/blendercam/releases

I used 3.0.2, which is the latest at the time of writing.

There is also a separate unstable repo, but it didn't work correctly for me at the time of writing: https://github.com/pppalain/blendercam/releases

I recommend renaming the downloaded zip to include the version, to avoid confusion.

## Install Blender

On Linux, you must download a zipped Blender from the official site, rather than install from the package manager, since the extension needs a specific Python version, which your distro might override with another version.

Download zipped Blender from here: https://download.blender.org/release/

For FabexCNC 3.0.2, Blender versions 5.1 and 5.0 didn't work. 5.1 straight up refused to enable the extension, and 5.0 enabled it, but then failed with missing Python functions during some operations. 4.5.x worked.

So download Blender 4.5.10 as an archive and unzip it.

## Enable FabexCNC

Go to `Edit`->`Preferences`->`Get Extensions` and enable internet access. FabexCNC will download its dependencies via it.

Press the \[V] button and "install from disk" the zipped FabexCNC directory.

Restart Blender. Better do it twice, just in case.

If you then need to delete the addon, it's somewhere in `~/.config/blender`. Nuking the entire directory is an option, if you don't have any important Blender settings.

## How to use FabexCNC

### Import the model

Import your 3D model using File->Import. E.g. the `.stl` format works.

### Set the units to `millimeters`

Change it in the `Scene` tab on the right, open `Units` there.

Also change the grid to match, by pressing the \[V] button next to two overlapping circles in the top-right corner of the viewport, and changing grid scale to `0.001` there.

Also scale your model if needed. To do that, scale it using the `Item` vertical tab on the right edge of the viewport. If the vertical tabs are hidden, unhide them using the \[<] button to the right of the XYZ axes widget.

### Operate FabexCNC

Open the `Render` tab in the bottom-right. Set the renderer to FabexCNC.

Set Interface->Complete to see more settings.

#### Set the machine settings

In the Machine section at the bottom, set everything:

* Post Processor = grbl
* Units = mm
* Work Area = ...
* Feedrate = ...
* Spindle speed = ... (don't forget Start Delay there, e.g. 3 seconds to let the spindle spin up)

Lastly, save this as a machine preset. When reusing the preset, check that all settings got restored!

#### Add operation

In the Operation section, press \[+] to add an operation (the lower of the two \[+] signs, not the one that adds a preset).

Important! Set the `Object` for this operation to your imported model. It defaults to the "cam machine" internal object, which is wrong.

Adding the operation unhides the Material tab in the bottom-right. Set your stock dimensions here.

Then configure the operation using the `CNC` vertical tab to the right of the viewport.

Set:

* Strategy = parallel

* Scroll to the bottom and configure the Cutter. Then save it to a preset, but note that not all settings seem to get saved.

  You likely want the Ballcone cutter type (which is the cone mill with a ball end).

* Go back to the top. Set Stepover and Detail (detail shouldn't be larger than Stepover probably).

  There is a warning if Stepover is larger than half the radius of the ballcone mill, so half is probably the good default. (Check the Engagement indicator lower in this window. It should be <= 50%.)

* Set Z Clearance to something smaller, like 2 mm.

* Set Operation Depth -> Max, if needed. You can set it to Custom and then type any value. The value should probably be negative, if your model is below zero as it should.

* If you don't want to mill out the entire cube, set Ambient -> Around, and specify some small radius.

* If you don't want multiple layer passes, uncheck Layers.

  In general, if you want to mill too deep, it seems you should first mill using a regular endmill, in multiple passes (as usual, each layer should be 1..2 diameters of the mill), with Skin > 0 (which grows the model to allow for a finishing pass later, this setting is at the top). And only then do the final pass with a ballcone mill, with no layers. Doing multiple ballcone mill layers only doesn't seem to work too well.

* In the Feedrate tab, set Feedrate and Spindle RPM again (they aren't pulled from the machine settings for me for some reason).

* In the Optimisation \[sic] tab, enable Exact Mode (for better quality). In there:

  * Enable `Use OpenCAMLib` (this seems to make the calculations much faster)

  * Enable Simplify G-code (the tutorial I used recommended this)

* In the Movement tab:

  * Enable Parallel Step Back to mill each line again when moving back (instead of raising the mill and moving over the top), thich results in better finish and shouldn't slow the operation down much if at all.

  * **Important:** Enabling `Stay Low (if possible)` seems to produce shorter gcode, but it can bug out and travel through your model. Check in the simulation.

* In the Operation G-code tab:

  * Enable Output Trailed, set it to `M5`. Otherwise it won't stop the spindle when done.

#### Generate G-Code

**Important!** In the scene tree, create a collection named `Paths`. Otherwise your resulting GCode will be silently missing after the generation.

Now is a good time to save your file in Blender. Firstly because the g-code will be generated in the same directory. And secondly because if the generation takes too long, you might want to kill the Blender process, and it'll be convenient to restart from here.

Click `Calculate Path & Export Gcode` on the right. It will show some progress in the status bar and sometimes as a progress bar in the top-right corner too.

This will compute the code but not export it. After this, an `Export Gcode` button will appear.
