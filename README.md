# Hackintosh-X79-Hobby

Hi everyone! One evening I was sitting at my PC and thought, "what if...". And that "what if" became a Hackintosh on an X79 motherboard.

## PC Specifications

| Component | Model / Details |
| :--- | :--- |
| **Motherboard** | X79 V2.72B (Sandy Bridge-E support) |
| **CPU** | Intel Xeon E5-2650 v2 @ 2.6 GHz |
| **RAM** | 16 GB 1600 MHz DDR3 |
| **GPU** | NVIDIA GeForce GTX 980 4GB |
| **Audio Codec** | Realtek ALC662 |
| **SMBIOS** | Mac Pro (2019) |

## Prelude
A quick background about me: I'm the developer of a game called [Daisen](https://store.steampowered.com/app/3702380/Daisen/). I recently took some time off for a vacation, and instead of just relaxing, I decided to mess around and build a Hackintosh just for fun.
Also, i have a 2011 iMac and use macOS Monterey on it with OCLP (OpenCore Legacy Patcher) so something i understand in Hacintosh.

## How I installed it

First of all: the EFI and getting to the macOS install screen. My CPU and motherboard didn't want to work with the EFI I created using OpenCore-Simplify. I spent a full day trying to make them work together, but nothing. The PC would either reboot or just drop a kernel panic. Interestingly, with the `cpus=1` boot arg, it worked... it worked. Then I found some random EFI and gave it a try. It worked, but not fully. For starters, that EFI didn't contain a kext for the CPU time fix, like CpuTscSync or VoodooTSCSync that can make some problems. 
***Info: CPU time fix (or Time Stamp Counter fix) is a kext that helps fix the CPU time counter for macOS. The main problem it solves is an incorrect time counter working async across cores.*** 
***Info 2: Kexts are like drivers for macOS. But while normal drivers tell your system how to work with your hardware, a kext is often trying to lie to your system.*** 

When I installed CpuTscSync, I just got a kernel panic. 'Kay, whatever. I installed the legacy VoodooTSCSync instead (I read on forums that VoodooTSCSync might actually be more stable for this). I also needed to edit `Info.plist` to set the thread count for my Xeon. When I tried to boot into the install screen again, the UI showed up, but the system dropped after maybe 10 seconds and rebooted. 
The fix: enable `DummyPowerManagement`. I also think the problem might be in the EFI's default `SSDT-PLUG`, so in the future, I'll need to generate my own with SSDTTime.

'Kay, after that I finally started installing macOS Monterey. I used an SSD I pulled from my laptop. When it installed and I started setting up the system... yeah, yeah, everything was SO laggy. That's 'cause of my Nvidia GPU.

Now for some more details: you see, Apple doesn't officially support Nvidia GPUs (or anymore). Yeah, there are some Mac mods that install better Nvidia GPUs, but it requires changing some software parts. So, if I wanted to fix the GPU, I needed a patch. I used OCLP and OCLP-Mod (the Chinese OCLP fork) and patched it. Now I have blur support and it works better. But! GPU acceleration didn't fully work. Why??? 'Cause Nvidia, what else. Old iMacs (pre-2012) have this exact same problem. Why? Metal API. You know that guy? That guy made everything even worse. Apple MAYBE uses Metal for rendering in macOS, which causes this whole mess. But it's usable, so why not. At least 144Hz works! :D

Next, I tried to connect to Wi-Fi. Ahem... my Realtek 8812CU (or tk-925ac) wasn't supported, not even by chris1111's drivers. I bought some Realtek-type USB Wi-Fi adapter on AliExpress. But while waiting for it to arrive, I still needed an internet connection. I tried using USB tethering, but Android hotspots don't work natively on macOS. To fix that, I installed a kext called HoRNDIS, and boom, everything worked. But! From what I understand, this only works up to macOS 12. If you try Ventura (macOS 13) or higher, it can't be used.

And my final boss was Audio. Well... I was trying to set up audio from 12:00, and it only finally worked at maybe 23:40. I'm trying EVERYTHING. And problem fully fixed when i RANDOMLY think to disable "Above 4G Decoding" (or 64-bit PCI Decoding) in the BIOS. And OMFG finally! It worked with VoodooHDA. But after that i returned to AppleALC and it also working if using `alcid=15` for my ALC662. I also patched HPET, but when I tried it, it didn't do anything for me.

So now I have a working macOS Monterey on my X79 motherboard. 
Why did I do all that? 
IDK for sure :b

## Useful Links & Tools

If you want to try building a Hackintosh on an X79 platform yourself, here are the essential resources and tools that helped me:

### The Arsenal
* **[Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)** - The ultimate, must-read step-by-step guide for creating your OpenCore EFI from scratch.
* **[OpCore-Simplify](https://github.com/lzhoang2801/OpCore-Simplify)** - Take it if you want to save yourself some time building the base EFI folder. But beware: it's not a magic "fix-all" button. It gives you a great starting point, but as my X79 experience showed, you'll still need to get your hands dirty and tweak things (like adding VoodooTSCSync or fixing SSDTs) to actually make it boot.
* **[OCAuxiliaryTools (OCAT)](https://github.com/ic005k/OCAuxiliaryTools)** - Basically a lifesaver for editing your `config.plist`. Forget trying to read and edit raw XML files in a text editor like me when I was trying to make my own EFI. OCAT gives you a clean GUI to tweak your boot args (like throwing in that `alcid=15` or `cpus=1`), update your kexts with a few clicks, and it even has a built-in validator to yell at you if you messed up something stupid in your config. Highly recommended for fine-tuning.
* **[USBToolBox](https://github.com/USBToolBox/kext)** - USB bind become simpler, nothing more.
* **[Hackintool](https://github.com/headkaze/Hackintool)** - Once you finally boot into macOS, you open this bad boy to see what actually works and what is still completely broken. It shows all your PCIe devices, helps check if your audio layout is recognized, rebuilds the kext cache, and checks your NVRAM. Also can map your USB :b
* **[OpenCore Legacy Patcher (OCLP)](https://dortania.github.io/OpenCore-Legacy-Patcher/)** - Essential for running newer macOS versions on unsupported SMBIOS and patching legacy Nvidia GPUs (like my GTX 980).
* **[How install HoRNDIS kext](https://youtu.be/32lM27TGNFM?si=oGdGx8LW1vbcmbDJ)** - Thanks man for the internet.

### Kexts (Drivers & Fixes)
* **[CpuTscSync](https://github.com/acidanthera/CpuTscSync)** - Crucial for fixing the Time Stamp Counter (TSC) sync issue on X79 motherboards and Xeon CPUs. Prevents kernel panics and lagging.
* **[ALT: VoodooTSCSync](https://github.com/RehabMan/VoodooTSCSync)** - When the modern CpuTscSync just gave me a kernel panic, this legacy kext actually worked! Just remember you have to manually edit its `Info.plist` to match your CPU's thread count (set `IOCPUNumber` to your total threads minus 1, so it's 15 for my 16-thread Xeon).
* **[AppleALC](https://github.com/acidanthera/AppleALC)** - The standard kext for native macOS audio support. For the Realtek ALC662, try boot arg `alcid=15`.
* **[VoodooHDA](https://sourceforge.net/projects/voodoohda/)** - A fallback audio driver. Good to have if AppleALC completely refuses to work on your specific Chinese X79 board.
* **[HoRNDIS](https://github.com/jwise/HoRNDIS)** - Android USB tethering driver for macOS. Very helpful if your Wi-Fi card isn't supported yet (Note: works only up to macOS Monterey).
* **[Lilu](https://github.com/acidanthera/Lilu)** - The "mother" of all kexts. Without this one, nothing else (like AppleALC or WhateverGreen) will work. It's the base for everything.
* **[VirtualSMC](https://github.com/acidanthera/VirtualSMC)** - Emulates a real Mac's SMC chip. It’s the modern standard that replaced the old FakeSMC.
* **[WhateverGreen](https://github.com/acidanthera/WhateverGreen)** - Fixes graphics issues, black screens, and port problems. Absolute must-have for my GTX 980 to even show a picture.
* **[RealtekRTL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X)** - The most common driver for Gigabit Ethernet on Chinese X79 boards.
* **[USBInjectAll](https://github.com/Sniki/OS-X-USB-Inject-All)** - Usually used during installation to "force-open" all USB ports so your mouse and keyboard actually work. 
* **[SMCProcessor & SMCSuperIO](https://github.com/acidanthera/VirtualSMC/releases)** - Part of the VirtualSMC package. These allow you to actually see how hot your Xeon is and how fast your fans are spinning.

### DSDT/SSDT Tools
* **[SSDTTime](https://github.com/corpnewt/SSDTTime)** - A simple script to dump your DSDT and generate custom, clean SSDTs (like `SSDT-PLUG`, `SSDT-EC`) directly from Windows or Linux.
