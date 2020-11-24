# Hackintosh on Asus ROG Maximus X Code via [OpenCore][OC]

![About this mac][System Info]

*macOS Supported:* **10.14+**

This is light configuration to run macOS smoothly. I didn't get any [kernel panics][uptime] science after macOS install. This config is base on [OpenCore Vanilla Desktop Guide][Guide].

## Hardware configuration

* Intel Core i7 8700K (5GHz OC) UHD630 (Geekbench 5: [CPU][GB_CPU]/[OpenCL][GB_OCL]/[Metal][GB_MTL])
* Asus ROG Maximus X Code
* 2×16GB Crucial Ballistix LT 3200MHz (3600MHz@cl16)
* M.2 NVME MyDigitalSSD SBX 120GB (macOS Catalina)
* SATA Micron M600 512GB (macOS Mojave)
* Dell Wireless 1820A (DW1820A) 802.11ac WLAN + Bluetooth 4.1 (DW1820A, P/N: CN-0VW3T3, 14e4:43a3/106b:0023)

## Before you start make sure you have

* Working hardware
* [BIOS][BIOS] version `>= 2102`
* Actual [OpenCore][OC] `= 0.6.3`
* Populated `PlatformInfo > Generic` section in `config.plist`, can be easyly done with `macserial` tool from [OpenCore][OC] utilities.

# Installation

## BIOS Settings

* *Exit* → Load Optimized Defaults [Yes]
* *Advanced* → System Agent (SA) Configuration → VT-d [**Enabled**]
* *Advanced* → System Agent (SA) Configuration → Primary Display [**IGPU**]
* *Advanced* → System Agent (SA) Configuration → DVMT Pre-Allocated [**64M**]
* *Advanced* → USB Configuration → Legacy USB Support [**Disabled**]
* *Advanced* → CPU Configuration → Intel Virtualization Technology [**Enabled**]
* *Boot* → Fast Boot [**Disabled**]
* *Boot* → CSM (Compatibility Support Module) → Launch CSM [**Disabled**]

## What's behind the scenes

You must download all not bundled kexts and drivers from repositories by yourself.

### Kexts

* Legacy_USB3.kext - Plist-only kext for USB port mapping
* [IntelMausi.kext][IntelMausi] - Another intel driver for Ethernet
* [AppleALC.kext][AppleALC] - Getting audio to work as easy-peasy
* [Lilu.kext][Lilu] - Dependency of `VirtualSMC.kext` and `WhateverGreen.kext`
* [VirtualSMC.kext][VirtualSMC] - A advanced replacement of FakeSMC, almost like native mac SMC.
* [WhateverGreen.kext][WG] - Need for GPU support (even for disabling discrete GPU)
* [AirportBrcmFixup.kext][WIFI] - Loader for `com.apple.driver.AirPort.Brcm4360` driver for wifi
* [BrcmFirmwareData.kext][BT] - Firmwares for various bluetooth devices
* [BrcmPatchRAM2.kext][BT] - Patch bluetooth for OS < 10.15
* [BrcmBluetoothInjector.kext + BrcmPatchRAM3.kext][BT] - Bluetooth patch and firmware injector for OS >= 10.15

### EFI drivers

* ~VirtualSMC.efi~ - only needed if you use File Vault 2 or [authrestart][FV2].

## Issues

1. The limit of USB ports is `15` but it counts not only physical but also protocol based. So if one physical port can be used by two protocols such as 3.0 (SS) and 2.0 (HS), in this way in system he actually own two of fifteen addresses (eg. HS01/SS01). You can see the real USB mapping in this [picture][USB map]. Due to these limits I disable a internal `HS12` which utilized by AURA Sync. Also internal `HS11` port is utilized by Bluetooth from m.2 NGFF wifi+bt combo module. And keep in mind USB 3.1 ports such as Type-C, Type-A and header provided by ASMedia controller.
2. Important! In `SSDT-EXT.aml` (using [MaciASL][MaciASL]), please, replace `#a` value for key `brcmfx-country` with your [country code][ISO_county_codes] for compliant with your country frequency limitations.

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
###### 24/11/2020
* HDMI video out fixed, I put HDMI cable to wrong port (DP) xD
###### 13/11/2020
* Update config for OpenCore v0.6.3
###### 5/10/2020
* Update config for OpenCore v0.6.2
###### 8/9/2020
* Update config for OpenCore v0.6.1
###### 14/8/2020
* Minor changes in SSDT-EXT.aml
###### 4/8/2020
* Update config for OpenCore v0.6.0
* Changed mac model to iMac19,1
###### 17/7/2020
* Updated config
* Updated SSDT
* Revert `Legacy_USB3.kext` naming
* macOS update to 10.15.6
###### 10/7/2020
* Changed mac model to iMac18,1
###### 7/7/2020
* Moved to macOS Catalina (10.15.5)
* High Sierra support dropped
* Add debug options
* Removed fake `device-id` for IGPU
* Added device properties for IGPU
###### 2/7/2020
* Fix ids for `UHD630`
* Move from `VoodooHDA` to `AppleALC`
###### 2/6/2020
* Update config for OpenCore v0.5.9
###### 22/5/2020
* Update config for OpenCore v0.5.8
* Removed `ApfsDriverLoader.efi`, already bundled in OpenCore v0.5.8
* Added `UEFI → APFS` section in `config.plist`
###### 23/4/2020
* Update config for OpenCore v0.5.7
* Removed DTGP method from `SSDT-EXT.aml`
* Updated references in `README.md`
###### 6/2/2020
* No changes, current config can be used with OpenCore v0.5.5
###### 10/12/2019
* Updated config complies with OpenCore v0.5.3
###### 9/11/2019
* Now using more efficient way to fix bluetooth with `BrcmBluetoothInjector.kext + BrcmPatchRAM3.kext`
###### 7/11/2019
* VoodooHDA tweaks
* Returning `AirportBrcmFixup.kext` to use `Brcm4360` instead of `BrcmNIC` due to several kernel panics
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

[AppleALC]: https://github.com/acidanthera/AppleALC
[BIOS]: https://www.asus.com/Motherboards/ROG-MAXIMUS-X-CODE/HelpDesk_BIOS/
[BT]: https://github.com/acidanthera/BrcmPatchRAM
[FV2]: https://lifehacker.com/bypass-a-filevault-password-at-startup-by-rebooting-fro-1686770324
[GB_CPU]: https://browser.geekbench.com/v5/cpu/4261705
[GB_MTL]: https://browser.geekbench.com/v5/compute/1671274
[GB_OCL]: https://browser.geekbench.com/v5/compute/1671254
[Guide]: https://dortania.github.io/OpenCore-Desktop-Guide/
[IntelMausi]: https://github.com/acidanthera/IntelMausi
[ISO_county_codes]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements
[Lilu]: https://github.com/acidanthera/Lilu
[MaciASL]: https://github.com/acidanthera/MaciASL
[OC]: https://github.com/acidanthera/OpenCorePkg
[System Info]: https://i.imgur.com/hgdywYt.png
[uptime]: https://i.imgur.com/OGl5UDI.png
[USB map]: https://i.imgur.com/eTJDKaB.jpg
[VirtualSMC]: https://github.com/acidanthera/VirtualSMC
[WG]: https://github.com/acidanthera/WhateverGreen
[WIFI]: https://github.com/acidanthera/AirportBrcmFixup
