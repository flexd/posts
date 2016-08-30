---
title: "The DIY Multimodule: Here is what you need to know"
slug: the-diy-multimodule-here-is-what-you-need-to-know
published: true
posted: 2016-08-30
updated: 2016-08-30
---

TL;DR:

* Solder things correctly fool! A few jumpers needs to be bridged for serial output.
* Make sure you buy a USBASP programmer. You need to flash the latest firmware! Get a 10-pin and fiddle with it like me, or 6-pin and connect directly.
* The default DSM option only enables 4 channels, and we need at least 6 if we want to control arming and flight modes with some switches as well. Choose the correct option (6 for 6 channels@11ms)
* Flashing on Linux: avrdude is your dude. Quick and easy :)

I've been flying my tiny little Hubsan X4 quad for at least a year, replacing motors, propellers and even the frame as they broke. And that was fine, but I wanted to take the step up to something bigger and better.
A proper big quad costs a lot of money, and also requires a lot of stuff that also costs money. I was trying to keep my costs down, so I decided I would just build a small brushed quad I could use to start out flying FPV, and hopefully avoiding crashing the bigger more expensive ones into the ground too much once I got one. The parts for my bigger quad are mostly here or being shipped here as I write this, so I guess I'll be documenting the build on that some time soon as well.

There are a lot of small brushed flight controllers out there, I'm not going to list them all since [Oscar Liang already has](https://oscarliang.com/brushed-micro-quad-parts-list/)
I chose to get a [Micro Scisky](http://www.rcgroups.com/forums/showthread.php?t=2466286) board, because it's based on the STM32 processor, which means it can run Cleanflight/Betaflight. So it's a good entry point for learning to setup a Cleanflight based system without having a bigger quad. It would also allow me to fly in rate/acro mode, which is what you do on all the proper quads, something that isn't possible on a hubsan.

The only problem is that the stock TX module in my Turnigy 9X radio is not capable of communicating with the FC, which speaks the DSM2/DSMX-protocol.

I could either buy a [Orange DSMX/DSM2 compatible module](http://www.hobbyking.com/hobbyking/store/__46634__OrangeRX_DSMX_DSM2_Compatible_2_4Ghz_Transmitter_Module_JR_Turnigy_.html), or I could find another solution.
I was on the fence about keeping my 9X radio at all, considering buying the popular [Taranis X9D](http://www.frsky-rc.com/product/pro.php?pro_id=113) that everyone else seems to use. But, that would mean not using a perfectly good radio I already had, and while the 9X has it's quirks, I have already spent a bunch of money improving it buying a [9XTreme board](http://smartieparts.com/shop/index.php?main_page=product_info&products_id=378) from SmartieParts, allowing me to run ersky9x, which is much better than the stock firmware.

I was talking about it with someone at the local hackerspace and they suggested I buy a `multi module`, a TX capable of talking many different protocols and frequencies. You can read more about the module [here](https://github.com/pascallanger/DIY-Multiprotocol-TX-Module) on GitHub, and [here](http://www.rcgroups.com/forums/showthread.php?t=2165676) on RCGroups.

You can either build it yourself, or it's available from [Banggood](http://www.banggood.com/2_4G-CC2500-A7105-Flysky-Frsky-Devo-DSM2-Multiprotocol-TX-Module-With-Antenna-p-1048377.html). I ordered mine from BG May 19th, and got it 11 days later. Which is surprisingly fast, it's shipped all the way from China.

You can find designs for 3D printing a case on [Thingiverse](http://www.thingiverse.com/thing:1691786), so that it fits nicely into the back of any radio with JR modules. It's probably possible to purchase these somewhere as well.

This post is really about my experience with this module, and what you need to look out for, I just wanted to give it a bit of a introduction.

After receiving the module I tried binding it with the Hubsan FC, and the Micro Scisky, but neither would bind.
I was trying to bind with the module in PPM mode, because I had heard that the modules banggood ship out had really old firmware, and the firmware in my 9X was too new so they could no longer communicate due to a protocol change.

**Banggood sells these with really old firmware !! If you are buying one of these modules, also buy a [USBASP](http://www.fischl.de/usbasp/) programmer, so you can flash the latest firmware!**

Unless you already have one, order one at the same time you order the Multi module. You will need it to flash new firmware onto the device. Make sure you either get one with a 6-pin cable, or you can fiddle like me with a 10-pin one and some wires. It's doable, but with a 6-pin you could just connect it directly. [Here](http://www.learningaboutelectronics.com/Articles/Program-AVR-chip-using-a-USBASP-with-6-pin-cable.php) is a website with a 6-pin diagram, and [here](http://www.learningaboutelectronics.com/Articles/Program-AVR-chip-using-a-USBASP-with-10-pin-cable.php) is the same website with a 10-pin diagram. Using those two you can easily see what needs to be connected where. Make sure you get it right, I did it entirely mirrored the first time compared to what I should have done. It took me a while to figure that out.

I have one of [these](http://www.banggood.com/USBASP-USBISP-3_3-5V-AVR-Downloader-Programmer-With-ATMEGA8-ATMEGA128-p-934425.html), it doesn't really matter where you get it from as long as it supports 3.3V

**When you are soldering the jumpers together, make sure you do it right and you have a good solder connection**
<img src="https://camo.githubusercontent.com/6ae6bd764cf916f7f959e22051e369c0b8b3a4be/687474703a2f2f7374617469632e726367726f7570732e6e65742f666f72756d732f6174746163686d656e74732f342f382f332f352f382f342f61393230363231372d3137372d494d475f353739302e6a7067"></img>

As the picture says. Solder those two jumpers, so that they look like they do in the picture. I did a bad job here, and noticed after a lot of debugging that one of the jumpers was not connected properly, and that's one of the reasons why it was not working. [This](https://github.com/pascallanger/DIY-Multiprotocol-TX-Module#buy-a-ready-to-use-and-complete-multi-module) is the relevant section of the README
It's not hard, just use a small tipped soldering iron (or try with your huge tip, good luck!), and a tiny bit of solder to bridge them. Make sure you do not bridge them sideways, it's easy if you add too much solder.

**Flashing the firmware on Linux? Use avrdude**
I just used `avrdude` here on Debian, it wasn't complicated and you need no drivers.
```
sudo avrdude -c usbasp -p m328p -u -U flash:w:/home/kristoffer/Downloads/Multiprotocol_V1.1.3_A7105-CC2500-CYRF6936.hex
```
[This](https://github.com/pascallanger/DIY-Multiprotocol-TX-Module#upload-the-code-using-isp-in-system-programming) is the relevant section of the README
That'll do the trick most likely. If you have connected it wrong, you'll spend a lot of time getting a error message. The warning about the usbasp firmware being out of date can safely be ignored. :-)

After doing all this, I still couldn't bind with anything. It turns out I had the protocol selector dial 180 degrees wrong. So it wasn't at 0 like I thought, but at 7 or 8.
**Make sure the protocol selector switch is at 0, and not 180 degrees wrong**


There was another problem though, even though I had configured one of the switches on the controller to channel 5 (AUX1), the flight controller never saw this.

But looking closer at the [DSM2 section of the documentation](https://github.com/pascallanger/DIY-Multiprotocol-TX-Module/blob/master/Protocols_Details.md#dsm2), I saw that you could change how many channels the module uses with the option setting.
I had not touched that, so it was on the default option 0, which means 4 channels at 22ms frame rate. 4 channels is not enough for A E T R (Aileron, Elevron, Throttle and Rudder) + AUX1 and AUX2. I have mine set to option 6 now, which is 6 channels @ 11ms framerate.

After changing that Cleanflight picked up the AUX channels just fine, and I was able to make one of the switches ARM the quad. 

Now it works perfectly, and I am able to fly my Hubsan and the Micro Scisky. I have not done a range test yet, since I mostly fly indoors or right outside the house, but for things like that it's brilliant!
