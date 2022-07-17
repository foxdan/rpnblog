---
layout: post
title: My understanding and summary of ECU fundamentals
---
This post is a copy from an old gist I wrote up a year or so ago. It's my basic
understanding from a software engineers PoV of what's going on in a very simple
ECU for a naturally aspirated petrol engine you might find in a motobike or an
older car. Some thoughts around tuning and so on.

Outputs
-------
* Fuel ("MAP")
* Spark (Ignition timing/advance)


Inputs
------
Can be some of or all of the following. In simplest form a single 2D array is
indexed by "load" and RPM, values represent the duration injectors are switched
on for. What is generally referred to as closed-loop means the value of the
Lambda sensor is taken into consideration when calculating the injector pulse
length. In open-loop the value of the Lambda sensor is ignored.

In a "standard basic" ECU we can assume it has two maps, a closed-loop map for
when the slope of the rate of change with regard to "load" and RPM is close to
zero, and open loop as it wavers. This largely because the response time of the
Lambda sensor is long. "Smarter" ECUs will use sampled recent Lambda values as
a factor in the open-loop calculation.

* Throttle (TPS)
* Crank (RPM)
* Air Temp (IAT)
* Barometric pressure (MAP)
* Air flow (MAF)
* Unburned fuel (Lambda/o2)


Fuel Tuning
-----------
When modifications are made to the stock internal combustion system it _may_ be
necessary to change the behaviour of the fuel injection system. The factory
closed-loop system will usually attempt to run the engine on the lean side of
the stoichiometric ratio for both fuel efficiency and emissions. The majority
of user modifications made will be within the margins the ECU will compensate
for. Noticeable problems will start to occur however under open-loop operation
if the ECU is not of the "learning" type that uses recent Lambda values for
open-loop.

### IAT Adjustment
A very simple solution here is to make some adjustments to the reported value
of the current air temperature.

* Beer-mat-maths
  * -10°C = +3% Fuel

The actual end effect of this input adjustment will of course vary across ECUs
and their underlying calculations. If it is constantly adjusting a factor where
Lambda takes priority, the effect may even be nil.

### Piggyback ECU
This is what a DynoJet Power Commander and similar devices are. These are
man-in-the-middle devices. In their absolute most basic form they will "trim"
the fuel injectors by running them for a longer/shorter period. To work
consistently they should also modify the ECUs reading of the Lambda sensor.
How these devices will usually work is not by modulating the output of the
Lambda but by providing values to keep it in a constant sate. So in effect
these devices keep an ECU in an open-loop type state with a modified map.

If we are building our own MITM device anything here goes of course with what
inputs & outputs we decide to modulate.

### ECU Flashing
Also referred to as "chip-tuning", "remapping", and so on. This process
overwrites the original program on the ECU supplied by the manufacturer. This
could be merely a modification to the lookup tables or it could be an entire
system change. This type of modification service is far too often a snake-oil
product sold to unsuspecting customers. Any idiot can get their hands on the
right hardware & software along with enough information to be dangerous here.

**DO NOT** engage in this kind of nonsense unless you have a full known good
(tested that you can restore it) backup of your original ECU.

"Ah but I know a guy" - Shit, you're right. Psst here's a another secret, [free
RAM](https://www.downloadmoreram.com).


Ignition Tuning
---------------
Adjusting the ignition advance for your combustion is a genuine way to more
power and efficiency. It needs to be tuned to the specific type of fuel you're
using. This is what usually makes the large power gains you get from a ECU
re-flash. But guess what there might be a reason it wasn't shipped from the
factory like that. If you now run an incompatible fuel you run the risk of
detonation. Unless you're screening your fuel somehow this is going to end in
a blown up engine.

