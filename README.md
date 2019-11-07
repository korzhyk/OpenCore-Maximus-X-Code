# Hackintosh on Asus ROG Maximus X Code via [OpenCore][1]

<p align="center">
  <img src="https://i.imgur.com/YNuOG3x.png" alt="About this mac specs">
</p>

*Mojave nVidia webdriver not avaliable yet :cry:* :idea: But you can use a High Sierra, why not?
This is light configuration to run macOS smoothly. I didn't get any kernel panics science after macOS install. This config is base on [OpenCore Vanilla Desktop Guide][10].

## Hardware configuration

* Intel Core i7 8700K (5GHz OC)
* Asus ROG Maximus X Code
* 2×16GB Crucial Ballistix LT 3200MHz (3600MHz@cl16)
* M.2 NVME MyDigitalSSD SBX 120GB (macOS)
* Dell Wireless 1820A (DW1820A) 802.11ac WLAN + Bluetooth 4.1 (DW1820A, P/N: CN-0VW3T3, 14e4:43a3/106b:0023)

## Before you start make sure you have

* Working hardware
* Last motherboard BIOS version
* Fresh OpenCore with filled `PlatformInfo > Generic` section in config.plist *if you are using a external GPU use mac Model 18,3 in another cases use 18,1*

# Installation

## BIOS Settings

* *Exit* → Load Optimized Defaults [Yes]
* *Advanced* → System Agent (SA) Configuration → VT-d [**Enabled**]
* *Advanced* → System Agent (SA) Configuration → Primary Display [**PEG**]
* *Advanced* → System Agent (SA) Configuration → DVMT Pre-Allocated [**64M**]
* *Advanced* → USB Configuration → Legacy USB Support [**Disabled**]
* *Advanced* → CPU Configuration → Intel Virtualization Technology [**Enabled**]
* *Boot* → Fast Boot [**Disabled**]
* *Boot* → CSM (Compatibility Support Module) → Launch CSM [**Disabled**]

## What's behind the scenes

You must download all not bundled kexts and drivers from repositories by yourself.

### Kexts

* USBROGMaximusXCode.kext - Plist-only kext for USB port mapping
* [IntelMausi.kext][8] - Another intel driver for Ethernet
* [VoodooHDA.kext][2] - Getting audio to work as easy-peasy
* [Lilu.kext][3] - Dependency of `VirtualSMC.kext` and `WhateverGreen.kext`
* [VirtualSMC.kext][4] - A advanced replacement of FakeSMC, almost like native mac SMC.
* [WhateverGreen.kext][5] - Need for GPU support
* [AirportBrcmFixup.kext][11] - Loader for `Brcm4331` driver for wifi
* [BrcmFirmwareData.kext + BrcmPatchRAM2.kext][12] - Firmware and patch for bluetooth fix

### EFI drivers

* ApfsDriverLoader.efi - Must have to run 10.13.6+
* FwRuntimeServices.efi - Must have to work with native NVRAM
* ~VirtualSMC.efi~ - only needed if you use File Vault 2 or [authrestart][6].

All drivers are from [AppleSupportPkg][9], there also contains driver for HFS+ `VBoxHfs.efi`

## Issues

1. The limit of USB ports is `15` but it counts not only physical but also protocol based. So if one physical port can be used by two protocols such as 3.0 (SS) and 2.0 (HS), in this way in system he actually own two of fifteen addresses (eg. HS01/SS01). You can see the real USB mapping in this [picture][101]. Due to these limits I disable a internal `HS12` which utilized by AURA Sync. Also internal `HS11` port is utilized by Bluetooth from m.2 NGFF wifi+bt combo module. And keep in mind USB 3.1 ports such as Type-C, Type-A and header provided by ASMedia controller.
2. Important! In `config.plist`, please, replace `#a` value for key `brcmfx-country` with your [ISO 3166-1 alpha-2 country code][13] for compliant with your country frequency limitations.

## USB ports mapping

| USB 2.0   |          | USB 3.0   |          |
|-----------|----------|-----------|----------|
| Port NAME | Port ADR | Port NAME | Port ADR |
| HS03      | 03       |           |          |
| HS04      | 04       |           |          |
| HS05      | 05       |           |          |
| HS06      | 06       |           |          |
| HS07      | 07       |           |          |
| HS08      | 08       |           |          |
| HS09      | 09       |           |          |
| HS10      | 0a       |           |          |
| Headers   |          |           |          |
| HS01      | 01       | SS01      | 11       |
| HS02      | 02       | SS02      | 12       |
| HS13      | 0d       |           |          |
| HS14      | 0e       |           |          |
| Internal  |          |           |          |
| HS11      | 0b       |           |          |
| ~HS12~    | ~0c~     |           |          |

Note: As you can see only two ports are avaliable on Super-Speed (USB 3.0) and this is front USB 3.0 header. But keep in mind, USB 3.1 provided by ASMedia also avaliable (rear Type-C, Type-A and front panel header).

## Chnagelog
###### 7/11/2019
* VoodooHDA tweaks
* Returning `AirportBrcmFixup.kext` due to several kernel panics on `BrcmNIC`
* Cleanup drivers and kexts, must be downloaded separately
###### 30/10/2019
* Looks WIFI module works OOB for this devid=14e4:43a3 with `BrcmNIC` driver
* Removed `AirportBrcmFixup.kext`
###### 28/10/2019
* Replaced Realtek wifi combo module with Dell Wireless 1820A
* Set `csr-active-config` to disable `SIP`
* Updated BIOS to version 2203
###### 11/10/2019
* Fixed HECI device
* Fixed \_DSM method injection
* Fixed SAT0 rename patch
* Updated device models in SSDT
###### 10/10/2019
* Removed custom DSDT
* Updated config
* Increase iMix and PCM in VoodooHDA.kext
* Fixed resolution while booting
###### 08/10/2019
* Update config to OC v0.5.1
* Removed `AppleGenericInput.efi`, already bundled in OpenCore v0.5.1
###### 07/10/2019
* The initial push to GitHub

[1]: https://github.com/acidanthera/OpenCorePkg
[2]: https://sourceforge.net/projects/voodoohda/
[3]: https://github.com/acidanthera/Lilu
[4]: https://github.com/acidanthera/VirtualSMC
[5]: https://github.com/acidanthera/WhateverGreen
[6]: https://lifehacker.com/bypass-a-filevault-password-at-startup-by-rebooting-fro-1686770324
[7]: https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/config.plist/coffee-lake#platforminfo
[8]: https://github.com/acidanthera/IntelMausi
[9]: https://github.com/acidanthera/AppleSupportPkg
[10]: https://khronokernel-2.gitbook.io/opencore-vanilla-desktop-guide/
[11]: https://github.com/acidanthera/AirportBrcmFixup
[12]: https://github.com/acidanthera/BrcmPatchRAM
[13]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements

[101]: https://i.imgur.com/eTJDKaB.jpg

