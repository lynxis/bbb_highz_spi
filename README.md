# Beagle Bone Black - changing high-z on spi if unused

## Why do I need this?

Imagine you connect the SPI pins to a Flash which is soldered
on a board (In-Cuircut).
As long the **target** board is offline, you can access the Flash via SPI.
But if you turn the **target** board online, you would have 2 Master on a SPI
bus connected to a single Slave. If you're lucky, the result is broken data.
But if you change the BeagleBoneBlack's SPI pins to High-Z, you don't stress the SPI bus and the **target** board can use the SPI as usual.

**picture**

## Compiling dts to dtbo files

You usually have a dtc compiler on your beagleboneblack.

```
dtc -@ -O dtb -I dts spidev_spi0.dts > /lib/firmware/LX-SPIDEV0-00A0.dtbo
dtc -@ -O dtb -I dts enable_spi0.dts > /lib/firmware/LX-ENABLE-SPI0-00A0.dtbo
dtc -@ -O dtb -I dts disable_spi0.dts > /lib/firmware/LX-DISABLE-SPI0-00A0.dtbo
```

## How can I put the SPI pins into High-Z?

It's important to know *device-tree-overlays*. It's a type of configuration of board. *device-tree overlays* can be loaded at runtime.
To access the bus you would load the files as folows:

 * connect spi engine to /dev/spidevX.0 (X can change)
   * `echo LX-SPIDEV0 > /sys/devices/platform/bone_capemgr/slots`
 * connect the SPI pins to the spi engine
   * `echo LX-DISABLE-SPI0 > /sys/devices/platform/bone_capemgr/slots`
 * disconnect the SPI pins from the spi engine and put them into High-Z
   * `echo LX-ENABLE-SPI0 > /sys/devices/platform/bone_capemgr/slots`

At boot time you configure the SPI engine to supply a spidev device.
So before booting you would disconnect the SPI pins.
And before access the SPI via the *BeagleBoneBlack* you connect the SPI pins to the spi engine.


## FAQ

### What happens if I'm unlucky? And using the BBB and target board spi pins at the same time?

You could kill the SPI pins (clock, MOSI, CS) of the beagleboneblack or of the target board.
