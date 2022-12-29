---
title: "Replacing the Bose amplifier on a Mazda3"
draft: false
categories: ['notes']
tags: ['cars', 'repair', 'electronics']
description: "A guide diagnosing and replacing a Bose amplifier on a 2010-2013 Mazda 3"
date: 2019-05-20T00:00:00-06:00
lastmod: 2020-08-06T00:00:00-06:00
---

Shortly after buying my 2011 Mazda 3, my radio suddenly stopped producing sound. Aside from having no sound, the radio still worked fine as the in-car Bluetooth and all the buttons still operated as normal. After doing a little research online, I found out that the amplifier for the Bose sound system died. I also found out that and that Bose replaces the defective amplifiers for free!

This article exists to help you diagnose the problem and deal with Bose yourself, so you do not have to go to a repair shop and have them fix it. Doing this replacement cost me absolutely nothing. In fact, I actually listed the old, non-working amplifier on eBay, and it sold for $50!

Keep in mind that the amplifier may not be your problem. If you simply lost sound like I did, it could be a wiring fault or something as simple as a blown fuse. However, if you hear a sort of machine gun sound [like this](https://www.youtube.com/watch?v=6LP-JGGeBuU), the problem is 100% the amplifier. If that is the case, you can simply skip the diagnosis and get a replacement.

## Finding the problem

To diagnose the amplifier, you will need:

1. A multimeter
2. 2 metal pins&mdash;[dressmaker pins](https://www.amazon.com/Singer-Dressmaker-Pins-500-Count-Size/dp/B000PSFC46) work great for this. These are used to get the multimeter probes connected to wiring harnesses.
3. Ratchet with a 10mm socket and if necessary, an extension

### Look for a blown fuse

First of all, with any electrical problem in your car, the first thing to look for is a blown fuse. To do this, open up the fusebox on the driver's side by the steering wheel. Look for a fuse labelled BOSE&mdash;it's one of the two 30A fuses. If it looks like the wire inside is broken, the fuse blew. Replace it. If you don't have power seats, such as in the i Touring trim level, you can use that fuse by pulling it out and putting it in the blown fuses' place.

If you hear sound now, there was your problem.

### Checking the wiring harness

If the fuse was not the culprit, you will want to see if you are getting +12V on two wires: remote turn-on and constant 12V. The constant 12V wire will be on at all times, whether or not your car is running or in accessory mode. The remote turn-on wire comes from your head unit (radio) and will only be supplying power if your radio is on. Note that it may not read 12V, it could read up to 13-14V if the engine is running.

Here is a wiring diagram for your amplifier connections: (you can click on it for a full-size photo)

<!--[![Bose pinout](/img/mazda-bose-pinout.jpg)](/img/mazda-bose-pinout.jpg)-->
![Bose pinout](/img/mazda-bose-pinout.jpg)

Start by connecting a multimeter to the **light blue** and **black** wires on the 8-pin connector. You should see 12V.

Now check the remote turn-on. On the 16-pin connector, connect the positive probe to the **light blue w/ black stripe** wire and the negative probe to the black wire on the 8 pin connector. You should see 12V with the radio powered on, and should read 0V with the radio off. On this wire you will want to use the dressmaker pins, as forcing the multimeter probes in can damage the amplifier connectors.

### Checking continuity of the speakers

Your amplifier could've gone into protect mode if a speaker is shorted out. Using the wiring diagram above, check the continuity of each speaker with a multimeter in the continuity function. Remember to connect positive to positive and negative to negative. Compare the impedance (resistance) of your speakers to this table below:

Speaker                 | Resistance
------------------------|-----------
Front center speaker    | 3.6 &Omega;
Front door speaker      | 2.15 &Omega;
Rear door speaker       | 3.4 &Omega;
Rear speaker            | 3.6 &Omega;
Subwoofer               | 1.0 &Omega;

If your amplifier is receiving power correctly and the speakers have correct impedances, your amplifier is dead.

## Removing the amplifier

First, slide the passenger seat all the way back. Lift up the flap in the carpet and you will see a 10mm bolt. Unscrew it.

![Front bolt](/img/mazda-bose-front.jpg)

Slide the passenger seat all the way forward.

![Rear bolts](/img/mazda-bose-rear.jpg)

Remove the black pastic caps over the 2 10mm nuts. Unscrew the 2 nuts. The amplifier assembly should now be loose.

Disconnect the 3 connectors on the amplifier and pull the amplifier assembly out from under the seat.

Finally, remove the 4 10mm nuts holding in the amplifier.

![Amplifier removed](/img/mazda-bose-amplifier.jpg)

## Contacting Bose

**Update 8/6/2020:** I have read somewhere that Bose has likely ended the free replacement amplifier program for the 2010-2013 Mazda3. You can try to contact Bose, but you may need to get your amplifier repaired or find a replacement elsewhere.

You will want to fill out [this form](https://automotive.bose.com/contact-us) on the Bose website. 

When filing a claim, you will need to gather and report the following information:

* Make, Model, Year
* VIN (take a picture of the label on the doorjamb)
* Amplifier part number and serial number
* Cloth or leather seats?
* Are you the original owner?

### Other replacement options

Assuming you can no longer get a replacement amplifier from Bose, there are some other options you can consider. One of these is getting your amplifier repaired using a repair service. These services usually offer a warranty on their work, so it will probably be your best bet.

Another option is buying a used amplifier from a salvage yard. This may be difficult, as the Bose package was only offered on certain trim levels of the Mazda3. Not to mention, you'll end up gambling on how long it will last.
