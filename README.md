# HP Pavilion Gaming 15-dk00xxxx Hackintosh OpenCore

**Status: Finished | Stable** <br>
**Current version: Ventura 13.7.4**

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.2-blue.svg)](https://github.com/acidanthera/OpenCorePkg)

## Introduction

**DISCLAIMER:**
Read the entire [Dortania](https://dortania.github.io/OpenCore-Install-Guide) guide before you start. I'm not responsible for any damage.
When you encounter bug or want to improve this repo, consider opening issue or pull request.
If you think that configuration is useful for you, consider to give a star for this repository :).

<details>
<summary>
    <strong>Hardware configuration</strong>
</summary>

### **HP Pavilion Gaming 15-dk00**


 | Component       | Manufacturer and model                                | Additional description           |
 | --------------- | ----------------------------------------------------- | -------------------------------- |
 | CPU             | Intel Core i5-9300H (9th gen - Coffee Lake Plus)      |                                  |
 | GPU             | Intel Graphics UHD 630                                |                                  |
 | External GPU    | NVIDIA GeForce GTX 1050 Graphics 3 GB GDDR5           | Disabled                         |
 | Screen          | 15.6" FHD IPS anti-glare (1920 x 1080)                |                                  |
 | RAM             | 24 GB DDR4 2667 MHz                                   |                                  |
 | SSD Primary     | GOODRAM IRDM PRO 240GB 2,5" SATA III                  | Disk for macOS and Manjaro Linux |
 | SSD Secondary   | Kingston A2000 (SA2000M8500G) 500GB M.2 NVMe          | Disk for Windows 10              |
 | Audio           | Realtek ALC285                                        |                                  |
 | Wireless        | Intel Wireless AX210                                  |                                  |
 | LAN             | Realtek RTL8168/8111 PCI-E Gigabit Ethernet Adapter   |                                  |
 | SD card reader  | Alcor Micro AU6625 PCI-E                              | Not working natively in macOS, see `Working with workarounds` section |
 | BIOS version    | F.40 Rev.A                                            |                                  |

</details>  

<details>
<summary>
    <strong>Kernel extensions</strong>
</summary>

All kext in use are up to date for 2025, in this project you can install mac os seqouya 15 but the wireless may stoped working you need to re configure it by 
using another method.

</details>

<details>
<summary>
    <strong>UEFI drivers</strong>
</summary>

|     Driver      | Version           |
| :-------------: | :---------------: |
| OpenHfsPlus.efi | OpenCorePkg 0.6.6 |
| OpenCanopy.efi  | OpenCorePkg 0.6.6 |
| OpenRuntime.efi | OpenCorePkg 0.6.6 |

</details>

## Recommended UEFI/BIOS settings

<details>  
<summary>
    <strong>UEFI/BIOS Setup</strong>
</summary>

<summary>
    <strong>Security</strong>
</summary>

- `Intel Software Guard Extensions (SGX) -> Enable`
- `TPM Device -> Enable`

<summary>
    <strong>Configuration</strong>
</summary>

- `Virtualization Technology -> Enabled`
- `Hyper-Threading -> Enabled`

<summary>
    <strong>Boot Options</strong>
</summary>

- `Legacy Support -> Disabled`
- `Secure Boot -> Disabled`

</details>

## Configuration advices (config.plist)

You can find configuration guide for Coffee Lake Plus laptops on [dortania.github.io](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#starting-point) site.
Below I presented details of config.plist that allow me to run system without problems
(after system installation). The most properties are the same as in Dortania guide,
but I also need to change some of them due to problems with booting of system installer.

<details>
<summary>
    <strong>ACPI</strong>
</summary>

- **Add**
  - Patches recommended via Dortania guide:
    - `SSDT-AWAC.aml`
    - `SSDT-EC-USBX-LAPTOP.aml`
    - `SSDT-PLUG-DRTNIA.aml`
    - `SSDT-PNLF-CFL.aml`
    - `SSDT-XOSI.aml`

  - Additional patches:
    - `SSDT-GPRW.aml` - instant wake fix patch
    - `SSDT-dGPU-Off.aml` - disable of dedicated NVIDIA GPU

- **Patch**
  - Change _OSI to XOSI:
    - `Comment -> Change _OSI to XOSI`
    - `Enabled -> True`
    - `Count -> 0`
    - `Limit -> 0`
    - `Find -> 5F4F5349`
    - `Replace -> 584F5349`
 
  - Change Method(GPRW,2,N) to XPRW, pair with SSDT-GPRW.aml:
    - `Comment -> change Method(GPRW,2,N) to XPRW, pair with SSDT-GPRW.aml`
    - `Enabled -> True`
    - `Count -> 0`
    - `Limit -> 0`
    - `Find -> 47505257 02`
    - `Replace -> 58505257 02`

</details>

<details>
<summary>
    <strong>Booter</strong>
</summary>

**Note**: In Quirks section I need to set different values than in Dortania guide for 
`DevirtualiseMmio`, `RebuildAppleMemoryMap` and `SyncRuntimePermissions` due to problems with booting (kernel panic).

- **Quirks**
  - `AllowRelocationBlock -> False`
  - `AvoidRuntimeDefrag -> True`
  - `DevirtualiseMmio -> False`
  - `DisableSingleUser -> False`
  - `DisableVariableWrite -> False`
  - `DiscardHibernateMap -> False`
  - `EnableSafeModeSlide -> True`
  - `EnableWriteUnprotector -> False`
  - `ForceExitBootServices -> False`
  - `ProtectMemoryRegions -> False`
  - `ProtectSecureBoot -> False`
  - `ProtectUefiServices -> True`
  - `ProvideCustomSlide -> True`
  - `ProvideMaxSlide -> 0`
  - `RebuildAppleMemoryMap -> False`
  - `SetupVirtualMap -> True`
  - `SignalAppleOS -> False`
  - `SyncRuntimePermissions -> False`

</details>

<details>
<summary>
    <strong>DeviceProperties</strong>
</summary>

- **Add**
  - Audio support
    - `PciRoot(0x0)/Pci(0x1F,0x3)`
      - `layout-id -> 0B000000`
      - `alc-delay -> 1000`

  - IGPU support
    - `PciRoot(0x0)/Pci(0x2,0x0)`
      - `AAPL,ig-platform-id -> 0900A53E`
      - `device-id -> 9B3E0000`

</details>

<details>
<summary>
    <strong>Kernel</strong>
</summary>

- **Quirks**
  - `AppleCpuPmCfgLock -> False`
  - `AppleXcpmCfgLock -> True`
  - `AppleXcpmExtraMsrs -> False`
  - `AppleXcpmForceBoost -> False`
  - `CustomSMBIOSGuid -> False`
  - `DisableIoMapper -> True`
  - `DisableLinkeditJettison -> True`
  - `DisableRtcChecksum -> False`
  - `ExtendBTFeatureFlags -> False`
  - `ExternalDiskIcons -> False`
  - `ForceSecureBootScheme -> False`
  - `IncreasePciBarSize -> False`
  - `LapicKernelPanic -> True`
  - `LegacyCommpage -> False`
  - `PanicNoKextDump -> True`
  - `SetApfsTrimTimeout -> -1`
  - `ThirdPartyDrives -> False`
  - `XhciPortLimit -> True`

 **Note**: `LapicKernelPanic -> True` is recommended for HP systems (according to Dortania guide).

</details>

<details>
<summary>
    <strong>Misc</strong>
</summary>

- **Boot**
  - `ConsoleAttributes -> 0`
  - `HibernateMode -> None`
  - `HideAuxiliary -> True`
  - `LauncherOption -> Disabled`
  - `LauncherPath -> Default`
  - `PickerAttributes -> 1`
  - `PickerAudioAssist -> False`
  - `PickerMode -> Builtin`
  - `PickerVariant -> Auto`
  - `PollAppleHotKeys -> False`
  - `ShowPicker -> False`
  - `TakeoffDelay -> 0`
  - `Timeout -> 0`

**Note**: I use rEFInd bootloader to select proper OS and in OpenCore I need only default system boot entry of macOS.
If you want to have selection of entries in OpenCore set `HideAuxiliary -> False`, `ShowPicker -> True`
and value of Timeout > 0 eg. `Timeout -> 10` (10 seconds).

- **Debug**
  - `AppleDebug -> False`
  - `ApplePanic -> True`
  - `DisableWatchDog -> True`
  - `DisplayDelay -> 0`
  - `DisplayLevel -> 2147483650`
  - `SerialInit -> False`
  - `SysReport -> False`
  - `Target -> 3`

- **Security**
  - `AllowNvramReset -> True`
  - `AllowSetDefault -> True`
  - `ApECID -> 0`
  - `AuthRestart -> False`
  - `BlacklistAppleUpdate -> True`
  - `DmgLoading -> Signed`
  - `EnablePassword -> False`
  - `ExposeSensitiveData -> 6`
  - `HaltLevel -> 2147483648`
  - `PasswordHash -> <>(empty value)`
  - `PasswordSalt -> <>(empty value)`
  - `ScanPolicy -> 0`
  - `SecureBootModel -> Default`
  - `Vault -> Optional`

</details>

<details>
<summary>
    <strong>NVRAM</strong>
</summary>

    LegacyEnable -> False
    LegacyOverwrite -> False
    WriteFlash -> True

- **Add**
  - System Integrity Protection bitmask
  `7C436110-AB2A-4BBB-A880-FE41995C9F82`
    - `boot-args -> keepsyms=1 -igfxblr`
    - `prev-lang:kbd -> 656E2D55 533A30`

   **Note**: For `boot-args` I added `-igfxblr` flag to prevent black screen on system loading screen.
   This problem appears after upgrading WhateverGreen kext version from 1.4.5 to 1.4.6. 
   Value for `prev-lang:kbd` enables English language for system installer.

</details>

<details>
<summary>
    <strong>PlatformInfo</strong>
</summary>

    Automatic -> True
    CustomMemory -> False
    UpdateDataHub -> True
    UpdateNVRAM -> True
    UpdateSMBIOS -> True
    UpdateSMBIOSMode -> Create
    UseRawUuidEncoding -> False

- **Generic**
  - `AdviseWindows -> False`
  - `MaxBIOSVersion -> False`
  - `ProcessorType -> 0`
  - `ROM -> 11223344 5566`
  - `SpoofVendor -> True`
  - `SystemMemoryStatus -> Auto`

 **Note**: You need to generate your own values for `SystemProductName`, `SystemSerialNumber`, `MLB` and `SystemUUID` using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
 I'm using SMBIOS for MacBookPro15.1, but in Dortania guide it's recommended to use SMBIOS for MacBookPro16.x
 (but when I using MacBookPro16.1 setup my bluetooth device was not recognized by system).

 **Note 2**: I provided random value for `ROM` section because for now I need only working Apple Store in my configuration.
 If you want to set up iMessage or iServices you can find dedicated [Dortania](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html) guide.

</details>

<details>
<summary>
    <strong>UEFI</strong>
</summary>

- **Quirks**
  - `DisableSecurityPolicy -> False`
  - `ExitBootServicesDelay -> 0`
  - `IgnoreInvalidFlexRatio -> False`
  - `ReleaseUsbOwnership -> True`
  - `RequestBootVarRouting -> True`
  - `TscSyncTimeout -> 0`
  - `UnblockFsConnect -> True`

 **Note**: `UnblockFsConnect -> True` is recommended for HP systems (according to Dortania guide).

</details>

## Status

<details>  
<summary>
    <strong>Not working ❌</strong>
</summary>

- `HDMI port`
  (but external display connection works, please see `Working with workarounds` section)
- `SD Card Reader`
  (but reading from and writing to SD Card works, please also see `Working with workarounds` section)

</details>

<details>  
<summary>
    <strong>Working with workarounds ⚠️</strong>
</summary>

- `External display connection with audio:`
  - There is no way to connect external display using HDMI or USB-C ports because there are paired with dedicated graphics card (NVIDIA GTX 1650) which is not supported by macOS higher than High Sierra.
  - `Workaround:` connection using laptop USB 3.0 port and [HDMI to USB 3.0 converter](https://www.cablecreation.com/pl/products/usb-adapter-cd0030.html),
  cost of device is around 30-40$ and you only need to install [DisplayLink](https://www.displaylink.com/downloads) driver to enable it.

- `SD card reading and writing:`
  - I cannot find way to enable Alcor Micro SD card reader (Alcor AU6625 PCI-E chip).
  - `Workaround:` Using USB 3.0 SD card reader. I'm using [Natec SCARAB](https://natec-zone.com/product/card-reader-natec-scarab-sd-micro-sd-usb-3-0-black), works out of the box.

</details>

<details>  
<summary>
    <strong>Working ✅</strong>
</summary>

- `App Store`
- `Audio` - Realtek ALC285 with sound keys (F7 and F8)
- `Brightness Keys` (reassignment to F2 and F3 keys is recommended)
- `Battery` (management, percentage and actual work time)
- `Bluetooth and Wi-Fi` - Intel Wireless-AC 9650
- `CPU power management / performance`
- `Ethernet port` - Realtek RTL8168/8111
- `Keyboard`
- `IGPU Intel UHD 630`
- `Internal microphone`
- `SATA SSD / NVMe support`
- `Shutdown / Reboot functions`
- `Sleep/Wake` - using Sleep from menu and after laptop lid close/open
- `Speakers and headphones combo jack`
- `System updates` (for now 2 updates for Big Sur were succesfully completed)
- `Touchpad`
- `USB Ports`
- `Web camera`

</details>

<details>  
<summary>
    <strong>Untested ❓</strong>
</summary>

- `iMessage, FaceTime, iTunes Store`
- `DRM`
- `Sidecar`
- `FireVault 2`

</details>

## Extras

This repository beyond Hackintosh EFI configuration also contains items related with macOS
(helpful scripts, boot manager configuration).

<details>  
<summary>
    <strong>Shutdown on low battery (SOLB) script</strong>
</summary>

Script based on [SleepOnLowBattery](https://www.tonymacx86.com/threads/release-sleeponlowbattery-solb.264785) from www.tonymacx86.com site.
Based script were written by users BugsB and Toggi3, huge thanks for your work.
My version is a little modification of script version without sound.

- **Features**:
  - `sleep command replaced with shutdown one`
  - `higher values of percents used to warn about low and very low battery`

It protected me several times from complete discharge of the battery :)

</details>

<details>  
<summary>
    <strong>rEFInd Boot Manager configuration</strong>
</summary>

I use 3 different operating systems on my laptop (macOS, Windows 10 and Manjaro Linux)
and it's necessary for me to select proper system to work on every boot.

I decided to use rEFInd Boot Manager due to problems with setup of Manjaro Linux entry
in OpenCore and it fully meets my expectations.

In extras/refind catalog you can find my refind configuration file (refind.conf)
and files for theme rEFInd-minimal-black (thanks for [@andersfischernielsen](https://github.com/andersfischernielsen/rEFInd-minimal-black) and [@EvanPurkhiser](https://github.com/EvanPurkhiser/rEFInd-minimal)).

</details>
