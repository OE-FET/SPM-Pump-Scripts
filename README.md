# SPM-Pump-Scripts
Scripts for the SPM syringe pump. Each script corresponds to a particular type of experiment.
The scripts are intended to be used with the custom "pump-specific" vi of the Agilent 415X driver, which combines the SPM pump with the 4156C SPA.


## Types of scripts:
**Priming** scripts: these are used to prime the tubings (fill them with liquid) prior to an experiment.

**Cycling** scripts: these are used to perform an experiment, usually by sequentially drawing liquids from many different reservoirs and pumping them into the sample.

**Cleaning** scripts: these are used to clean the tubings after an experiment.

## How to write cycling scripts:
-	A script is split into an integer number of **steps** depending on the number of liquids that are sequentially pumped into the sample. If water, buffer, and again water are pumped into the sample, then the script consists of three steps.
-	A step consists of an integer number of **cycles**. During a cycle, the pump draws liquid from a reservoir and pumps it into the sample. The cycles are identical, except for the following differences.
-	There should be only one initialization command `/1ZR` per script. This should be the first command of the first cycle. The next cycles should not have an initialization command.
-	The first cycle also should not have a valve port query `/1?6`. It is meaningless, since the pump is being initialized.

So, the way to write the cycling scripts is:
-	Write the first cycle.
-	Copy and paste it, as needed.
-	Remove the initialization commands from all cycles except the first, and the first valve port query from the first cycle (the /1?6 query that comes after the initialization command).


### Structure of each cycle:

1.	The pump is initialized (if in the first cycle).
2.	The step resolution is set.
3.	The (drawing) flow is set.
4.	The pump moves to the port of the reservoir.
-	NOTE: The pump should move clockwise when moving to a higher port number and vice versa.
-	NOTE: Every port move should be followed by a query and a  valve port query command.
5.	The pump draws a volume of liquid into the syringe.
a.	NOTE: The volume of liquid to be drawn should equal the pumping flow x measurement time. Otherwise, the pump will have no more liquid to pump into the device while the measurements are going on.
6.	A time delay command follows, to allow the pump enough time to draw the liquid into the syringe.
-	NOTE: The time delay should be at least equal to the volume of liquid to be drawn divided by the drawing flow.
7.	The (pumping) flow is set.
8.	The pump moves to the port of the device.
-	NOTE: The pump should move clockwise when moving to a higher port number and vice versa.
-	NOTE: Every port move should be followed by a query and a  valve port query command.
9.	The pump pumps the volume of liquid into the device.
-	NOTE: The volume of liquid to be drawn should equal the pumping flow x measurement time. Otherwise, the pump will have no more liquid to pump into the device while the measurements are going on.
10.	There is no time delay command, to allow the pump enough time to pump the liquid into the syringe. The liquid pumping is done during the measurements.
11.	The device is being measured, while the liquid is pumped in the device.
ii.	There are various query commands that are executed inbetween the above commands. There are:
1.	The query command `/1Q`.
a.	Every pump command (except other queries) should be followed by a query command. The status bit shows if the pump is busy only if the input command is a query (see manual).
2.	The plunger position query command `/1?`.
3.	The valve port query command `/1?6`.
-	NOTE: Every port move command should be followed by a query and a valve port query command. This is to record that the port switch actually took place. If the valve port is queried later, it will seem like the port switch took place later, which is false: it took place earlier, but there was no valve port query command to record it.
4.	Transfer/output curves on the device.
5.	The order of the query commands I use is:
-	Query command `/1Q`
-	Plunger position query command `/1?`
-	Valve port query command `/1?6`
-	Transfer curve

Note that the timing between the plunger position query commands and the valve port query commands is crucial!
-	The valve port query commands and the plunger position query commands should give an accurate impression of how much liquid was drawn from each reservoir and where it was pumped.
-	Every port move command should be followed by a query and a valve port query command. This is to record that the port switch actually took place. If the valve port is queried later, it will seem like the port switch took place later, which is false: it took place earlier, but there was no valve port query command to record it. This may create the impression that the liquid was drawn from the wrong reservoir.

7.	The time units of the LabVIEW script should match the time delays between the query commands. If LabVIEW is set to plot the valve port every minute and I have a script that queries the valve port every second, I will end up with multiple data points on the same time, which will not make sense.
8.	There is a command that changes a part of the filename. This is to mark different conditions to which the sample is subjected (i.e. to denote the change of the “step”). For example, if a WG-OFET has different liquids pumped into it, this command could be used to indicate which measurements belong to which liquid. In this case, the filename change command is inserted at the beginning of each step.

3.	How to write priming scripts:
-	Priming scripts are similar to cycling scripts but do not contain the device measuring part. The drawing and pumping flows are also much larger.
-	When drawing liquid into the syringe, the position of the plunger depends on the volume of the tubing. The syringe must draw enough liquid to fill the tubing.

4.	Port settings:
-	Port 1 should always be connected to the waste. During initialization, the pump will empty the contents of the syringe to port 1.
