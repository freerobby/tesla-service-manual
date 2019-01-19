---
layout: post
title:  "Replacing eMMC in MCU"
published: true
---

## Description

Tesla MCUs contain an eMMC (memory chip) that commonly fails after heavy write cycles. When this occurs, you will get a blank screen on your CID.

These instructions explain how to replace the eMMC chip rather than having to replace the entire MCU.
## Applies To

* All pre-refresh Model S vehicles


## Procedure

### Introduction
                      
The MCU ("Media control Unit") is the large center screen. Assuming you're pre-2016 the CID card is what runs the MCU OS. It's a daughtercard on the MCU mainboard which is made by nVidia.

The Tegra T3 is the main processor for the MCU and it's on the CID (the chip under the heatsink)... and inside the T3 is a boot coprocessor in addition to the main T3. On reset this boot coprocessor initializes to the first address in the boot flash, which is the Spansion flash on the obverse side of the CID. 

You'll find that this flash has much more capacity (512MB) than you'd normally expect for an embedded device, because this code is for much more than a skeleton boot. The coprocessor runs it, and reads the main OS from the Hynix eMMC flash chip on the front of the CID. This Tegra (Ubongo) Linux filesystem is written into RAM by the boot coprocessor. 

Once that's complete the coprocessor chains to the main processor which boots the OS in RAM, and then mounts /home and /var on partitions in the eMMC. It makes sense that they'd want /home and /var non-volatile, for car-specific functions, logs, etc.

Now, code-signing is used, which is started very early in the boot coprocessor's initialization, in what is effectively its BIOS. What this means is that if you manually downgrade the OS in the eMMC by writing with dd, you still have the boot code for the later version in the Spansion chip which doesn't match. The T3 so fails to boot due to code-signing failure and a black screen. So the active firmware in the eMMC must match the version in the boot flash -- do not unilaterally upgrade the active partition's firmware. Ask me how I know...

Tesla has extensive logging going on, for good reasons, although over time this will inevitably wear out the (low-quality) Hynix chip. SD/MMC chips not specifically named 'endurance' are not able to take too much writing. This means that when eMMC partition (/var) wears out, the T3 will fail to boot and black screen.
                      
### Approach

My solution was to have a phone shop unsolder my eMMC, put it in a special AllSocket chip carrier for 153 pad ball-grid array chips, dump the image and put this firmware image down on a high quality industrial-grade chip with far more capacity. Then I rooted it myself. I put a Netgear Power-over-Ethernet switch between the IC ('Instrument Cluster') and MCU, which has an ethernet connection between them, and connected a Tinker Board to the switch, for wifi and control. At some point I'll add fore and aft lipstick dashcams. (main reason I used a Tinker Board, quite a bit more powerful than a Raspberry Pi 3+)

If you have a black screen the good news is the damage is likely only to partition 3. (/var) And it is imperative that you run a release in your replacement eMMC which matches the version of your Spansion boot chip, due to code-signing. You have that, in partition 1 or 2 or both. So do the following steps carefully. With tech, a miss, is as good as a mile.

### Procedure

Order an Allsocket carrier like [this one](https://www.amazon.com/ALLSOCKET-eMMC169-Programmer-Kingston-Interface/dp/B06Y55DKND?tag=diyelectriccarconvert-20). 

You'll need a new eMMC. SD/MMC is an interface standard for the industry so that in theory any eMMC is a drop-in replacement for one of the same formfactor. I've looked everywhere for an 'endurance'-type eMMC chip but haven't found one yet, so the next best thing is a high-quality chip. Whereas the stock Hynix chip is a low-grade Hynix H26M42002GMR 8GB, I've replaced mine with a [SwissBit SFEM064GB1EA1TO 64GB](https://swissbit.com/products/nand-flash-products/managed-nand/e-mmc/). Moar storage is never a problem (with quality components). I bought mine at [Mouser Electronics](https://eu.mouser.com/new/Swissbit/swissbit-em-20-series-memory/). The new chip will be balled from the factory so you can pop it in the AllSocket (paying close attention to pin 1).

Take a picture of the Hynix chip before they remove it so you can identify pin 1, which isn't marked on this sorry chip. It helps them if you remove the shroud and heatsink from the T3. No big deal. Have the eMMC chip removed and reballed. You need a phone repair that does rework, with an infrared soldering workstation. They'll know exactly what to do, but don't tell them it's for your car... they may get frightened. 'It's for your stereo' or something. Hopefully they have a stainless steel mask (153 pad .5mm pitch) and solder balls, for reballing the pads, and hopefully they place pieces of metal over other components to protect them as they un/solder.

![Phone repair ad]({{ "/assets/replacing-emmc-in-mcu-phone-repair.jpg" | absolute_url }})

Put the chip in the AllSocket carrier, paying close attention to (your picture of) pin 1, and put that into your computer. Need to be running Linux, I recommend CentOS but any will do.

```
dmesg
```

... and the last few lines will show what the chip came up as, usually /dev/mmcblk0 and associated partitions p1-p4.

```
cd /home/{youruser}/dl
```

... or wherever you want to put the images. You first want to pull the full image, bit-for-bit as there are one or more blank blocks at the beginning and it is very important to preserve the file structure.

DO NOT MOUNT the partitions at this point.

```
dd if=/dev/mmcblk0 of=mmcblk0.img bs=4M
```

... substitute your input device if different. 'bs=' block-size just speeds up the transfer and makes no reference to structure. This is your Golden Image. Save it, save a backup of it, treasure it.

Now for safety and study, also preserve each partition:

```
dd if=/dev/mmcblk0p1 of=mmcblk0p1.img bs=4M
```

... note carefully how this command differs from the above. Do this also with p2 through p4.

It's possible you will get errors around partition 3, but do the best you can. You must get as much data as possible.

Now; on the Hynix chip it is possible your filesystem is damaged (due to worn-out memory cells), so let's check it:

```
fsck /dev/mmcblk0p1
```

... (no, not F*CK!, an old Linux joke), and do each through partition 4. If it finds damage, have it try to repair with

```
fsck -y /dev/mmcblk0p3
```

If you continue to have a problem with part 3's image we'll have to figure something else out. You don't want to mount and just copy the files because that would not preserve ownership, rights, and so on. `rsync --archive` would work.

If you got a clean full image from above, or have repaired part 3 and then gotten a clean full image, remove the Allsocket and the Hynix from within, and replace with your new eMMC.

```
dd if=mmcblk0.img of=/dev/mmcblk0 bs=4M
```

... you see, to Linux everything looks like a file, even devices, so you can treat them that way. 

If all has gone well, what we've done is lay down a bit-for-bit image of your old flash onto the new one. This means that the image is limited to 8GB, whereas your new chip has a capacity of 64GB. Fortunately Tesla made /home as the 4th partition, so let's expand that to take up the rest of the chip, giving us access to that space for future fun.

```
gparted
```

... make -sure- that the device selected in the upper-right is the mmcblk0, or else you're about to ruin your boot disk. Select the resize tool and drag part 4's partition to take up the rest of the chip. Check-mark. Close gparted and consider your next move.

Note: user verygreen offers this feedback:

> It will work better if you do NOT extend home to fill entire chip but leave some "unpartitioned" space instead. The unused space would work as a buffer and that helps reduce wear as the EMMC becomes full (since filesystem does not signal to it when the space is free it does not know what bits are unused and needlessly copying them around). By having more space than is actually used, write amplification is reduced as you approach full storage. With a 64G chip you can easily leave 32G unpartitioned for a great benefit but even 8G would do wonders. For more info search on "ssd write amplification"


Now would be a good time to root the chip and take other measures. That's a long story and I am time-limited.

For now you will be able to have the shop resolder the chip and it should boot. Whole chip rework process cost me $125. Hold off on this if you want to root though.

### Epilogue

I'm not going to put in one of those useless 'I am not responsible...' disclaimers, but needless to say this all involves risk. Don't even think about it if you don't have engineering and solid Linux skills, just pay somebody. I recommend ce2078 and verygreen, who will give -you- root and they will not monitor you, as opposed to those commercial rooters; be prepared to compensate them for their time.

There is also a hazard here if you wait to do this until you have a black screen. The eMMC wearing out is inevitable due to heavy logging. It is possible by this point that partition 3 (/var) is damaged and unreadable. It's best to do this before you have a problem. Although let's face it, that's not human nature. I don't know that you will recover if your chip is worn out.

Modifying the firmware, you can open the ethernet diag port, get root, and many other things. Optionally you can put a switch between the ICU and MCU -- I put in a [Netgear GS105PE](https://www.netgear.com/support/product/gs105pe), a PoE switch so I also needed to add a 12v-to-PoE converter. (eBay) Don't try to power the 'right' ethernet lines because PoE no longer works that way. 

Also to the switch I added a [Tinker Board](https://www.asus.com/us/Single-Board-Computer/Tinker-Board/), which is more powerful than a Raspberry Pi, since I plan to add cameras. The Tinker Board requires more current than the latest Pi so be sure to get a big enough 12v-to-5v buck power supply. I hated the TinkerOS ('Armbian') so I installed the latest Fedora/Arm with the efficient LXQt desktop. And I did the rubber-ducky antenna mod on the Tinker so it associates with my home Unifi Pro AP. 

Soon I'll be installing Wireguard in the CID, which is a next-gen VPN, so whatever IP the car gets on GSM it'll reach out to home and connect. I had been using IPSec and never learned OpenVPN because it's lame.

## References and Citations

This entry was documented by DIYElectricCar user Quantum, with an addendum by verygreen. The original version can be found [here](https://www.diyelectriccar.com/forums/showthread.php?p=1029731#post1029731).
