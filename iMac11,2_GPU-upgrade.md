# GPU Upgrade on an iMac11,2 (mid-2010, 21.5")
## Why
One day the screen of my trusty iMac11,2 didn't switch on any more. Curiously the Mac bootet fine - it chimed, fans spun up and if I waited the time it normally took to boot, I could hit `Esc` on the keyboard and would hear the MacOS error sound coming from the speakers. So the system seemed to work fine, just the screen stayed off. 

I checked whether it was the backlight by shining a bright light at the screen - it wasn't - and connected an external display to check whether the LCD was bad - it wasn't - that stayed dark too. So it must have been the GPU that went bad. That seems to happen quite often with iMacs of this generation. Their AMD GPUs just fail. 

In my case the failed GPU was the stock **AMD Radeon HD 4670**. (Amusingly, on the GPU itself its still branded as an ATI Radeon - different times!) Getting that exact GPU is quite difficult, they're expensive and - well - old, so even if I were to score a working HD 4670 who knows how soon that one would fail as well. That's when I fell into the rabbit hole that is the MacRumors [Thread](https://forums.macrumors.com/threads/2011-imac-graphics-card-upgrade.1596614/) on the topic. I decided to switch out my old broken HD 4670 against an **AMD M5100**. That GPU physically fits in the iMacs MXM GPU slot, only needs a minor mod to the cooling system and offers good compatibility with newer MacOS versions. The latter is due to the fact that the M5100 supports Metal accelleration - the HD 4670 doesn't. So I went for it.

## Procuring the AMD M5100
I ordered my card from eBay. I got a red AMD M5100 for around 40$ from a Chinese seller. The MXM-interface GPU was listed as new and while I can't be 100% sure, it came very neatly packaged and didn't display any signs of wear and tear. Maybe it was old new stock. 🤷

## Identifying the AMD M5100
These MXM GPUs are made for use in x86 gaming laptops. If you want to use them in an Apple iMac you have to flash a Mac-compatible vBIOS onto it. We'll get to that later, but it's important to know what variant of the card you have. Most common are M5100s with a red PCB, those are manufactured by Dell. Then there is the memory. Different red Dell cards come with memory from different manufacturers. Mine was a red Dell card with Hynix memory of the AFR variant. I noted that for the flashing that we'll get to later.

## Performing the operation itself
So now it was time to proceed to the actual GPU switch itself. All the steps needed to disassemble the iMac can be found [here](https://www.ifixit.com/Guide/iMac+Intel+21.5-Inch+EMC+2389+GPU+Card+Replacement/6272) on iFixit. 

* Crack open the iMac as described on iFixit.
* Disconnect the original SSD.
* Remove the original, broken GPU.
* (While I was inside the iMac, I also removed the CPU heatsink and applied new thermal paste. That was definitely a good idea, CPU temperatures are way lower since.)
* Install the new M5100.
	* My M5100 came with a preinstalled X-bracket that doesn't fit with the iMac's heatsink screws. I used acohol to loosen the adhesive of the preinstalled bracket and then carefully peeled it off the GPU. Then I installed the GPU with the Mac-original bracket.
	* The M5100's chip is a bit smaller than the one on the HD4670 - that means the heatsink won't make good contact with the new card, which will lead to a quick heat-induced death... So following the MacRumors *GPU Swap Bible* I installed a copper shim between the chip and heat sink.
	* I used a 0.5 mm thick copper plate. I cut it to size so it would fit exactly on top of the M5100's chip, coated both sides with thermal paste and sandwiched it between the chip and the heatsink. 
* That's it, I reassembled everthing, but left the LCD panel off as well as the SSD unplugged.

## Flashing

To flash the M5100 with the needed Mac-compatible vBIOS, I created a USB with Ausdauersportler's specialized [GRML-Linux](https://github.com/Ausdauersportler/GRML-FLASH). For it to boot I have to leave the LCD screen and the SSD disconnected, otherwise the iMac just gets stuck in some mysterious pre-boot stage. Once GRML has bootet I connect to it via ssh (I plugged in a network cable) and flash the GPU. The tool `amdvbflash` and the neccessary vBIOS are all present on GRML.

```
root@grml ..live/mount/persistence/sdb/flash/Video # ./amdvbflash -p 0 AMD/GCN1-3/M5100/M5100-GOP_HynixAFR.rom 
AMDVBFLASH version 4.71, Copyright (c) 2020 Advanced Micro Devices, Inc.

Old SSID: 15CC
New SSID: 15CC
Old P/N: BR44815.001
New P/N: BR46365.001
Old DeviceID: 6821
New DeviceID: 6821
Old Product Name: Venus XTGL C42251 GDDR5 128Mx16 OPM 
New Product Name: Venus XTGL C42251 GDDR5 128Mx16 OPM 
Old BIOS Version: 015.036.000.006.044815
New BIOS Version: 015.036.000.006.046365
Flash type: M25P10/c
Burst size is 256 
20000/20000h bytes programmed
20000/20000h bytes verified
```

Then restart System To Complete VBIOS Update, plug SSD and LCD back in and reset the P-RAM. (`alt, cmd, P und R`)

## Create an Installer

To create an installer I used the OCLP app on another Mac. I downloaded Monterey (Ventura doesn't support bluetooth on the iMac11,2 ATM) and wrote it to a USB drive. Then I created an OCLP config with "AMD GCN" set as GPU overrride under the "Advanced" tab and also installed that to the thumb drive. Now I could plug that in and after around 30 seconds the iMac screen came to life to show the OCLP boot selector. I selected the Monterey installer and after an hour of loading screens and reboots I was in! Now I just had to boot into the system. On first boot OCLP pops up and asks, whether I want to install it onto the SSD. I did that - no overrides or special settings required - and that was it! 

## Ressources
* [iMac11,2 on Everymac](https://everymac.com/systems/apple/imac/specs/imac-core-i3-3.06-21-inch-aluminum-mid-2010-specs.html)
* [MacRumors Thread: 2011 iMac Graphics Card Upgrade](https://forums.macrumors.com/threads/2011-imac-graphics-card-upgrade.1596614/)
* [iFixit disassembly guide](https://www.ifixit.com/Guide/iMac+Intel+21.5-Inch+EMC+2389+GPU+Card+Replacement/6272)
* [GRML-Linux to flash the GPU](https://github.com/Ausdauersportler/GRML-FLASH)

# Troubleshooting Zoom

At the time of writing there is a problem, where Zoom won't ask for the permission to use camera and microphone on OCLP-enabled versions of Monterey. (It's an "AMFI" problem apparently.) Use the tool [tccplus](https://github.com/jslegendre/tccplus/releases/tag/1.0) to fix that:

```
./tccplus add Microphone us.zoom.xos
./tccplus add Camera us.zoom.xos
```
 
 If you need to use this for a different app you can find the right app id (`us.zoom.xos` in this case )as follows: `codesign -dr - /Applications/zoom.us.app` Just drag and drop the app icon right after the second hyphen.

#### Sources
* [tccplus Github](https://github.com/jslegendre/tccplus/)
* [Stackexchange Guide](https://superuser.com/questions/1779925/macos-ventura-cant-give-permissions-opencore)

# Quick guide, [curtesy of Nguyen Duc Hieu on Macrumors](https://forums.macrumors.com/threads/2011-imac-graphics-card-upgrade.1596614/post-32232676)
```
iMac 27" 2009, High Sierra.
Dead HD4850 replaced by FirePro M6100

What I did:
Replaced the HD4850 with M6100
Boot the iMac with no SSD or HDD, no LCD, confirm that it could boot normally with a chime.
Do a PRAM reset.
Plug in the GRML Linux USB and reboot. Confirm all fans spin normally. GRML Linux USB was prepared previously on my Windows PC.

From another PC (I used Windows PC), SSH to the iMac and type necessary command to do the flash.

Take a blank SSD + USB enclosure, plug it to another working iMac of mine.
Clone High Sierra volume to the SSD
Install OCLP EFI on the SSD.

Transfer the SSD to iMac 2009 with M6100.
Assemble the LCD panel.

My iMac now boot normally to High Sierra.
```

