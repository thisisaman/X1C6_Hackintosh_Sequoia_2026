# ThinkPad X1 Carbon Gen 6 — macOS Sequoia EFI
### 🟢 Golden Build · Tested March 2026

> A fully functional OpenCore EFI for the **Lenovo ThinkPad X1 Carbon Gen 6 (2018) [20KG]**, tested on **macOS 15 Sequoia** with verified OTA upgrade support from Sonoma 14.x.

---

## ⚠️ Before You Start

**This EFI does not include a Serial Number.** You must generate your own SMBIOS data before booting. See [SMBIOS Setup](#2-smbios-critical) below.

---

## Hardware Specifications

| Component | Details |
|---|---|
| **Model** | Lenovo ThinkPad X1 Carbon Gen 6 (20KG) |
| **CPU** | Intel Core i5-8250U (8th Gen, Kaby Lake-R) |
| **iGPU** | Intel UHD Graphics 620 |
| **RAM** | 8GB LPDDR3 |
| **Display** | 14" 1080p IPS (Internal) |
| **External Display** | 1440p via HDMI (HiDPI verified) |
| **Audio** | Realtek ALC285 |
| **Wi-Fi / Bluetooth** | Intel Wireless-AC 8265 |
| **Bootloader** | OpenCore v1.0.6 |

---

## Compatibility Status

| macOS Version | Status |
|---|---|
| macOS 15 Sequoia (15.x) | ✅ Tested — Golden Build |
| OTA Upgrade from Sonoma 14.x | ✅ Verified |

---

## What Works

| Feature | Status | Notes |
|---|---|---|
| Sleep / Wake | ✅ | Native power management, perfect lid sleep |
| Clamshell Mode | ✅ | External monitor with lid closed |
| Trackpad | ✅ | Full VoodooI2C multi-touch gestures |
| TrackPoint | ✅ | |
| Keyboard | ✅ | All keys, backlight, display brightness & volume keys |
| HDMI | ✅ | Hot-plugging and audio out functional |
| Headphone Jack | ✅ | |
| Wi-Fi | ✅ | See [Wi-Fi & Bluetooth](#wi-fi--bluetooth) section |
| Bluetooth | ✅ | |
| Webcam | ✅ | |
| Microphone | ✅ | |
| USB-A Ports | ✅ | Custom USB map included |
| USB-C Ports | ✅ | |
| Power Management | ✅ | SMCProcessor + NVMEFix configured |
| Battery Status | ✅ | SMCBatteryManager |
| Wired Ethernet | ✅ | Intel I219-V via IntelMausiEthernet |
| ThinkPad Extras | ✅ | Fan control, airplane mode LED via YogaSMC |

## Not Tested

| Feature | Notes |
|---|---|
| microSD Card Reader | No results to report yet |
| Thunderbolt 3 | No results to report yet |

## Does Not Work

| Feature | Notes |
|---|---|
| Fingerprint Sensor | Not supported — no macOS driver exists for this hardware |
| iPhone Mirroring | Requires a T2 chip, which is not present on this machine |
| DRM (Apple TV+, Sidecar) | May require additional iGPU patches depending on display config — see [Known Issues](#known-issues) |

---

## Wi-Fi & Bluetooth

This EFI ships with **`itlwm.kext`** by default, which requires the **HeliPort** app to manage Wi-Fi connections.

- 📥 **HeliPort** (Wi-Fi menu bar app): [GitHub — OpenIntelWireless/HeliPort](https://github.com/OpenIntelWireless/HeliPort/releases)

Alternatively, if you prefer native Wi-Fi integration using `AirportItlwm`, refer to **EliteMacx86's** in-depth guide for Sequoia:
- 📖 [How to Fix Intel Wi-Fi and Bluetooth on macOS Sequoia and Later](https://elitemacx86.com/threads/how-to-fix-intel-wifi-and-bluetooth-on-macos-sequoia-and-later.2066/)

> **Note:** `AirportItlwm` may require a different kext build per macOS minor version. Always verify you have the correct build for your exact Sequoia version.

---

## Installation Notes

### 1. BIOS Settings

Access BIOS with **F1** at boot and configure the following:

**Disable:**
- Secure Boot
- Intel SGX
- Fast Boot

**Enable:**
- UEFI Boot Mode
- AHCI Mode
- VT-d

### 2. SMBIOS (CRITICAL)

This EFI targets **`MacBookPro15,2`** as its SMBIOS — the 2018 13" MacBook Pro, which shares the same CPU generation and is natively supported by macOS Sequoia.

You **must** generate your own unique identifiers before first boot:

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
2. Generate values for `MacBookPro15,2`
3. Populate the following fields in `EFI/OC/config.plist` under `PlatformInfo > Generic`:
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`

> ⚠️ Booting with an empty or duplicate serial number can cause iCloud and iMessage activation issues.

### 3. USB Mapping

The included `UTBMap.kext` is mapped for the standard **X1 Carbon Gen 6** internal hub configuration and covers:

- USB-A ports (left and right)
- USB-C ports
- Internal Bluetooth hub
- Internal webcam hub

If your **Bluetooth or Camera** does not appear after installation, your internal hub location may differ slightly. Use **Hackintool → USB** to identify the correct port addresses and regenerate the map.

---

## Known Issues

### DRM (Apple TV+, Sidecar)
Hardware DRM is limited on iGPU-only Hackintoshes. Apple TV+ may show a black screen or refuse to play. Sidecar functionality may vary depending on your display configuration. Some users have success with additional `shikigva` boot args via WhateverGreen — research based on your specific display setup.

### iPhone Mirroring
This feature is tied to the **T2 / Secure Enclave** chip and cannot be made to work on non-Apple hardware regardless of software patches.

---

## EFI Overview

```
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/              # DSDT patches (battery, sleep, etc.)
    ├── Drivers/           # OpenRuntime, HfsPlus, etc.
    ├── Kexts/             # All kexts listed below
    ├── Resources/         # OpenCanopy theme assets
    ├── Tools/
    └── config.plist
```

### Kexts Included

| Kext | Purpose |
|---|---|
| **Lilu** | Core patching foundation — required by most other kexts |
| **VirtualSMC** | SMC emulation — essential for macOS to boot |
| **WhateverGreen** | iGPU / display patching (framebuffer, HDMI) |
| **AppleALC** | Audio — ALC285 |
| **VoodooPS2Controller** | Keyboard & TrackPoint |
| **YogaSMC** | ThinkPad-specific features: fan control, airplane mode LED, keyboard shortcuts |
| **itlwm** | Intel Wi-Fi (use with HeliPort — see Wi-Fi section) |
| **IntelBluetoothFirmware** | Intel Bluetooth firmware upload |
| **IntelBTPatcher** | Bluetooth fix for macOS 12+ |
| **BlueToolFixup** | Bluetooth stack patch required for macOS 12+ |
| **IntelMausiEthernet** | Wired Ethernet (Intel I219-V) |
| **ECEnabler** | Expands EC fields to allow proper battery status reading |
| **SMCBatteryManager** | Battery percentage and status (VirtualSMC plugin) |
| **SMCProcessor** | CPU temperature and power reporting (VirtualSMC plugin) |
| **SMCSuperIO** | Fan speed monitoring (VirtualSMC plugin) |
| **SMCLightSensor** | Ambient light sensor (VirtualSMC plugin) |
| **NVMEFix** | NVMe power management and latency improvements |
| **BrightnessKeys** | Fn brightness key support |
| **USBToolBox** | USB map companion |
| **UTBMap** | Custom USB map (generated via USBToolBox) |

---

## Credits & Resources

This EFI would not exist without the incredible work of the Hackintosh community:

- **[Acidanthera](https://github.com/acidanthera)** — OpenCore, Lilu, WhateverGreen, AppleALC, and the broader kext ecosystem
- **[Dortania](https://dortania.github.io/OpenCore-Install-Guide/)** — The definitive OpenCore installation guide
- **[OpenIntelWireless](https://github.com/OpenIntelWireless)** — itlwm, AirportItlwm, and HeliPort
- **[EliteMacx86](https://elitemacx86.com/)** — In-depth Sequoia Wi-Fi/Bluetooth guide
- **[Rybo713](https://github.com/Rybo713)** — X1C6 community reference
- **[corpnewt](https://github.com/corpnewt)** — GenSMBIOS, ProperTree, and many essential tools
- **[VoodooI2C Team](https://github.com/VoodooI2C)** — Trackpad support

---

## Disclaimer

This EFI is provided as-is for educational purposes. macOS is proprietary software intended for Apple hardware. Use on non-Apple hardware is done entirely at your own risk. The author takes no responsibility for data loss, hardware damage, or any other issues arising from the use of this EFI.
