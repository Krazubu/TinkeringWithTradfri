# <a>Teardown guide</a>


The lightbulbs are rather easy to take apart, you can open them in a few minutes without breaking anything.
If done gently, you’ll even be able to reassemble them without anyone being able to notice.
This stands for the “classic” bulbs, as GU10 ones or less common shapes might be more difficult to open.

## 1 - Remove the bulb
<p align="center">
<img src="1.jpg" width="800">
</p>

The bulb is glued to the plastic body with some not so strong and gummy glue.
Use something like a butter knife, you don’t need something especially thin, avoid blades and knifes with teeth for both your security and to not scratch the parts.
Take the bulb horizontally and push the knife in the notch between the bulb and the body at a 90° angle.
Don’t try to go under the plastic yet, this might not be needed.
Rotate holding the bulb, holding the knife this way so you cut the glue all around.
Once done try to lift firmly the bulb from the base. The bulb doesn’t seem to be made of glass and is tolerant to some torsion, but don’t be too strong so it doesn’t explode inside your hand.
If it’s not enough, try to pry gently under the plastic body all around the bulb, you don’t have to go deep, it only  goes about 2mm under.
After that, the bulb should definitely come with no difficulty, and still no damage.


## 2 - Remove the plate

<p align="center">
<img src="2.jpg" width="800">
</p>

You can now see the plate holding the LEDs and the antenna of the zigbee module passing through a hole.
Remove the 2 phillips screws and unplug the plate by lifting evenly all around to not bend the plate or the pins.

## 3a - Remove the Zigbee module - Methode 1

<p align="center">
<img src="4.jpg" width="800">
</p>

Now you can see the inside electronics and the zigbee module which is fitted perpendicularly in a notch on the main PCB.


Here comes the chirurgical part.
Using a soldering iron, you should have enough room in the body to unsolder the zigbee module from the other side of the PCB, where the module goes past the surface about 1 or 2mm.
As there are numerous contacts on both sides, you’ll have to be precise and patient.
I took the bulb in my left hand, putting my index on the back of the zigbee module and applying a soft but constant pressure, to lift the module off the notch.
With the right hand, drag the tip of the soldering iron along the tiny angle formed by the 2 PCBs crossing each other (see photo for 3bis below).
Go slow enough so the solder has time to melt, and fast enough so you can reach the other end while it is still fluid. Putting some more tin may help, it will communicate heat better and will take longer to get hard. The contacts on the 2 sides are for the same tracks so you should not need to do it on both sides as heat will communicate from one side to the other. However if it was not the case don’t hesitate to alternate on both sides.
Drag like this several times, with your constant pressure, it will slowly progress out of the notch.
The notch is quite tight and adjusted, tilting the module may get it stuck, so take care to keep it in a straight position. When one side comes out more than the other, progress on the opposite side, and so on…
If you’re good at it, it should not take long.

## 3b - Remove the Zigbee module - Methode 2 (possibly destructive)

<p align="center">
<img src="5.jpg" width="800">
</p>

If you’re not comfortable/patient enough for the above methode, you can still remove the metallic screw on the bottom by unscrewing it from the plastic base. It is stamped at its top so it might be a bit difficult and damage the plastic.

## 4 - Wire the connections you need

<p align="center">
<img src="6.jpg" width="800">
</p>

Once you have your module, you can put it on a breadboard or whatever you want. Except for the right side, the pads around are really tiny, I used single strands of a multi strand wire to have wires tiny and flexible enough.
On the picture you can see the 4 top pads required for JTAG and SWO wired using strands of a bigger wire. The single wired pad on the left is the reset, and the 6 bigger pads on the bottom are +3,3V, GND and 4 GPIOs.
